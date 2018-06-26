# Message Handling for Committing a Block {#handling}

## [Types](handling_types.md#types) {#types}

## Walkthrough {#walkthrough}

1. `handleMsg()` reads the message from the peer; see [`eth/handler.go#L320`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L320):
    ```go 
    msg, err := p.rw.ReadMsg()
    ```

2. The message is of type `NewBlockMsg`, so the block data is decoded and scheduled for import:
[`eth/handler.go#L634-L664`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L634-L664)
    ```go
    // handleMsg is invoked whenever an inbound message is received from a remote
    // peer. The remote connection is torn down upon returning any error.
func (pm *ProtocolManager) handleMsg(p *peer) error {
    
    // snip  TODO this is a REALLY long method and should be refactored
    
    case msg.Code == NewBlockMsg:
        // Retrieve and decode the propagated block
        var request newBlockData                  
        if err := msg.Decode(&request); err != nil {
            return errResp(ErrDecode, "%v: %v", msg, err)
        }
        request.Block.ReceivedAt = msg.ReceivedAt
        request.Block.ReceivedFrom = p

        // Mark the peer as owning the block and schedule it for import
        p.MarkBlock(request.Block.Hash())
        pm.fetcher.Enqueue(p.id, request.Block)

        // Assuming the block is importable by the peer, but possibly not yet done so,
        // calculate the head hash and TD that the peer truly must have.
        var (
            trueHead = request.Block.ParentHash()
            trueTD   = new(big.Int).Sub(request.TD, request.Block.Difficulty())
        )
        // Update the peers total difficulty if better than the previous
        if _, td := p.Head(); trueTD.Cmp(td) > 0 {
            p.SetHead(trueHead, trueTD)

            // Schedule a sync if above ours. Note, this will not fire a sync for a gap of
            // a singe block (as the true TD is below the propagated block), however this
            // scenario should easily be covered by the fetcher.
            currentBlock := pm.blockchain.CurrentBlock()
            if trueTD.Cmp(pm.blockchain.GetTd(currentBlock.Hash(), currentBlock.NumberU64())) > 0 {
                go pm.synchronise(p)
            }
        }
        
        // snip
    }
    ```

3. The block fetcher then tries to import the new block (see `#3`); see
[`eth/fetcher/fetcher.go#L277-L314`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L277-L314). `Suggestion` refactor the `loop` method so it is not so long.
    ```go
    // Loop is the main fetcher loop, checking and processing various notification
    // events.
    func (f *Fetcher) loop() {
        // Iterate the block fetching until a quit is requested
        fetchTimer := time.NewTimer(0)
        completeTimer := time.NewTimer(0)
    
        for {
            // Clean up any expired block fetches
            for hash, announce := range f.fetching {
                if time.Since(announce.time) > fetchTimeout {
                    f.forgetHash(hash)
                }
            }
            // Import any queued blocks that could potentially fit
            height := f.chainHeight()
            for !f.queue.Empty() {
                op := f.queue.PopItem().(*inject)
                hash := op.block.Hash()
                if f.queueChangeHook != nil {
                    f.queueChangeHook(hash, false)
                }
                // If too high up the chain or phase, continue later
                number := op.block.NumberU64()
                if number > height+1 {
                    f.queue.Push(op, -float32(number))
                    if f.queueChangeHook != nil {
                        f.queueChangeHook(hash, true)
                    }
                    break
                }
                // Otherwise if fresh and still unknown, try and import
                if number+maxUncleDist < height || f.getBlock(hash) != nil {
                    f.forgetBlock(hash)
                    continue
                }
                f.insert(op.origin, op.block)  // <<=== #3
            }
        
        // snip
    ```

4. Check that the block header validates; see [`eth/fetcher/fetcher.go#L635-L682`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L635-L682)

    ```go 
    // insert spawns a new goroutine to run a block insertion into the chain. If the
// block's number is at the same height as the current import phase, it updates
// the phase states accordingly.
func (f *Fetcher) insert(peer string, block *types.Block) {
    hash := block.Hash()

    // Run the import on a new thread
    log.Debug("Importing propagated block", "peer", peer, "number", block.Number(), "hash", hash)
    go func() {
        defer func() { f.done <- hash }()

        // If the parent's unknown, abort insertion
        parent := f.getBlock(block.ParentHash())
        if parent == nil {
            log.Debug("Unknown parent of propagated block", "peer", peer, "number", block.Number(), "hash", hash, "parent", block.ParentHash())
            return
        }
        // Quickly validate the header and propagate the block if it passes
        switch err := f.verifyHeader(block.Header()); err {
        case nil:
            // All ok, quickly propagate to our peers
            propBroadcastOutTimer.UpdateSince(block.ReceivedAt)
            go f.broadcastBlock(block, true)                          // <<=== #4a

        case consensus.ErrFutureBlock:
            // Weird future block, don't fail, but neither propagate

        default:
            // Something went very wrong, drop the peer
            log.Debug("Propagated block verification failed", "peer", peer, "number", block.Number(), "hash", hash, "err", err)
            f.dropPeer(peer)
            return
        }
        // Run the actual import and log any issues
        if _, err := f.insertChain(types.Blocks{block}); err != nil {  // <<=== #4b
            log.Debug("Propagated block import failed", "peer", peer, "number", block.Number(), "hash", hash, "err", err)
            return
        }
        // If import succeeded, broadcast the block
        propAnnounceOutTimer.UpdateSince(block.ReceivedAt)
        go f.broadcastBlock(block, false)                

        // Invoke the testing hook if needed
        if f.importedHook != nil {
            f.importedHook(block)
        }
    }()
}
    ```
  a. If the block header validates correctly it is propagated to the node&apos;s peers (`#4a` above).

  b. The block is inserted into the forked blockchain (`#4b` above).

5. The block is processed by `insertChain` (`#5`); see [`core/blockchain.go#L1134-L1162`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L1134-L1162)
    ```go
    // insertChain will execute the actual chain insertion and event aggregation. The
    // only reason this method exists as a separate one is to make locking cleaner
    // with deferred statements.
    func (bc *BlockChain) insertChain(chain types.Blocks) (int, []interface{}, []*types.Log, error) {
   
     // snip  TODO this really long method should be refactored
    
        // Create a new statedb using the parent block and report an
        // error if it fails.
        var parent *types.Block
        if i == 0 {
            parent = bc.GetBlock(block.ParentHash(), block.NumberU64()-1)
        } else {
            parent = chain[i-1]
        }
        state, err := state.New(parent.Root(), bc.stateCache)
        if err != nil {
            return i, events, coalescedLogs, err
        }
        // Process block using the parent state as reference point.
        receipts, logs, usedGas, err := bc.processor.Process(block, state, bc.vmConfig)  // <<=== #5
        if err != nil {
            bc.reportBlock(block, receipts, err)
            return i, events, coalescedLogs, err
        }
        // Validate the state using the default validator
        err = bc.Validator().ValidateState(block, parent, state, receipts, usedGas)
        if err != nil {
            bc.reportBlock(block, receipts, err)
            return i, events, coalescedLogs, err
        }
        proctime := time.Since(bstart)

        // Write the block to the chain and get the status.
        status, err := bc.WriteBlockWithState(block, receipts, state)
    
    // snip
    }
    ```

6. If the block is processed successfully; see [`core/blockchain.go#L1134-L1162`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L1134-L1162)
    ```go
    // WriteBlockWithState writes the block and all associated state to the database.
    func (bc *BlockChain) WriteBlockWithState(block *types.Block, receipts []*types.Receipt, state *state.StateDB) (status WriteStatus, err error) {
        bc.wg.Add(1)
        defer bc.wg.Done()
    
        // Calculate the total difficulty of the block
        ptd := bc.GetTd(block.ParentHash(), block.NumberU64()-1)
        if ptd == nil {
            return NonStatTy, consensus.ErrUnknownAncestor
        }
        // Make sure no inconsistent state is leaked during insertion
        bc.mu.Lock()
        defer bc.mu.Unlock()
    
        currentBlock := bc.CurrentBlock()
        localTd := bc.GetTd(currentBlock.Hash(), currentBlock.NumberU64())
        externTd := new(big.Int).Add(block.Difficulty(), ptd)
    
        // Irrelevant of the canonical status, write the block itself to the database
        if err := bc.hc.WriteTd(block.Hash(), block.NumberU64(), externTd); err != nil {
            return NonStatTy, err
        }
        // Write other block data using a batch.
        batch := bc.db.NewBatch()
        rawdb.WriteBlock(batch, block)  <<=== #6a
    
        root, err := state.Commit(bc.chainConfig.IsEIP158(block.Number()))  // <<=== #6b
    ```
  a. The new block is written to the chain (`#6a` above)
  ```go
  // WriteBlockWithoutState writes only the block and its metadata to the database,
  // but does not write any state. This is used to construct competing side forks
  // up to the point where they exceed the canonical total difficulty.
  func (bc *BlockChain) WriteBlockWithoutState(block *types.Block, td *big.Int) (err error) {
      bc.wg.Add(1)
      defer bc.wg.Done()
    
      if err := bc.hc.WriteTd(block.Hash(), block.NumberU64(), td); err != nil {
          return err
      }
      rawdb.WriteBlock(bc.db, block)
    
      return nil
  }
  ```
  
  b. It is committed to the database (`#6b` above); see [`core/state/statedb.go#L581-L628`](https://github.com/ethereum/go-ethereum/blob/master/core/state/statedb.go#L581-L628).
  ```go
    // Commit writes the state to the underlying in-memory trie database.
    func (s *StateDB) Commit(deleteEmptyObjects bool) (root common.Hash, err error) {
        defer s.clearJournalAndRefund()
    
        for addr := range s.journal.dirties {
            s.stateObjectsDirty[addr] = struct{}{}
        }
        // Commit objects to the trie.
        for addr, stateObject := range s.stateObjects {
            _, isDirty := s.stateObjectsDirty[addr]
            switch {
            case stateObject.suicided || (isDirty && deleteEmptyObjects && stateObject.empty()):
                // If the object has been removed, don't bother syncing it
                // and just mark it for deletion in the trie.
                s.deleteStateObject(stateObject)
            case isDirty:
                // Write any contract code associated with the state object
                if stateObject.code != nil && stateObject.dirtyCode {
                    s.db.TrieDB().InsertBlob(common.BytesToHash(stateObject.CodeHash()), stateObject.code)
                    stateObject.dirtyCode = false
                }
                // Write any storage changes in the state object to its storage trie.
                if err := stateObject.CommitTrie(s.db); err != nil {
                    return common.Hash{}, err
                }
                // Update the object in the main account trie.
                s.updateStateObject(stateObject)
            }
            delete(s.stateObjectsDirty, addr)
        }
        // Write trie changes.
        root, err = s.trie.Commit(func(leaf []byte, parent common.Hash) error {
            var account Account
            if err := rlp.DecodeBytes(leaf, &account); err != nil {
                return nil
            }
            if account.Root != emptyState {
                s.db.TrieDB().Reference(account.Root, parent)
            }
            code := common.BytesToHash(account.CodeHash)
            if code != emptyCode {
                s.db.TrieDB().Reference(code, parent)
            }
            return nil
        })
        log.Debug("Trie cache stats after commit", "misses", trie.CacheMisses(), "unloads", trie.CacheUnloads())
        return root, err
    }
  ```
