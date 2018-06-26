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

3. The block fetcher then tries to import the new block; see
[`eth/fetcher/fetcher.go#L313`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L313). `Suggestion` refactor the `loop` method so it is not so long.
    ```go
    f.insert(op.origin, op.block)
    ```

4. If the block header validates correctly it is propagated to the node's peers; see [`eth/fetcher/fetcher.go#L654-L657`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L654-L657)
    ```go 
    go f.broadcastBlock(block, true)
    ```

5. The block is inserted into the forked blockchain; see [`eth/fetcher/fetcher.go#L669-L672`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L669-L672)
    ```go
    if _, err := f.insertChain(types.Blocks{block}); err != nil {
          glog.V(logger.Warn).Infof("Peer %s: block #%d [%x…] import failed: %v", peer, block.NumberU64(), hash[:4], err)
          return
    }
    ```

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
    