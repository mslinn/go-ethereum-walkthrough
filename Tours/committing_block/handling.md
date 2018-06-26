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

6. The block is processed; see [`core/blockchain.go#L1147`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L1147)
    ```go
    receipts, logs, usedGas, err := bc.processor.Process(block, state, bc.vmConfig)
    ```

7. If the block is processed successfully it is committed to the database; see [`core/blockchain.go#L902`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L902)
    ```go
    root, err := state.Commit(bc.chainConfig.IsEIP158(block.Number()))
    ```

8. The new block is written to the chain; see [`core/blockchain.go#L900`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L900)
    ```go
    rawdb.WriteBlock(batch, block)
    ```
    