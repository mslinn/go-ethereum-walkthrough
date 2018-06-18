# Following a Block From the Socket Until Committed to the Blockchain

_This is a modified version of [go-ethereum-code-walkthrough.md](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [gsalgado](https://github.com/gsalgado). Code references were updated, more detail was added, links to types were provided and code snippets were added._

Note: All links to the code are based on `master` as it was when this document was last modified. Line numbers might shift as a result.

## Preconditons
* `geth` is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected to a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## Initialization

1. `geth` sets up a full [`Node`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L40-L74); see [`cmd/geth/main.go#L236`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L236):<pre>// geth is the main entry point into the system if no special subcommand is ran.
// It creates a default node based on the command line arguments and runs it in
// blocking mode, waiting for it to be shut down.
func geth(ctx *cli.Context) error {
        node := makeFullNode(ctx)
        <b>startNode(ctx, node)</b>
        node.Wait()
        return nil
}</pre>

2. `geth` registers an instance of the [`eth.Ethereum`](https://github.com/ethereum/go-ethereum/blob/master/eth/config.go#L76-L117) service with that `Node`; see [`cmd/geth/config.go#L156`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L156):<pre>func makeFullNode(ctx *cli.Context) *node.Node {
        stack, cfg := makeConfigNode(ctx)
        <b>utils.RegisterEthService(stack, &cfg.Eth)</b>
        if ctx.GlobalBool(utils.DashboardEnabledFlag.Name) {
            utils.RegisterDashboardService(stack, &cfg.Dashboard, gitCommit)
   }</pre>

3. A [`Protocol`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L25-L55) `struct` is created for every supported protocol when `geth` starts (the startup sequence is not shown here): <pre>// Protocol represents a P2P subprotocol implementation.
type Protocol struct {
       // Name should contain the official protocol name,
       // often a three-letter word.
       Name string
       // Version should contain the version number of the protocol.
       Version uint
       // Length should contain the number of message codes used
       // by the protocol.
       Length uint64
       // Run is called in a new groutine when the protocol has been
       // negotiated with a peer. It should read and write messages from
       // rw. The Payload for each message must be fully consumed.
       //
       // The peer connection is closed when Start returns. It should return
       // any protocol-level error (such as an I/O error) that is
       // encountered.
       Run func(peer *Peer, rw MsgReadWriter) error
       // NodeInfo is an optional helper method to retrieve protocol specific metadata
       // about the host node.
       NodeInfo func() interface{}
       // PeerInfo is an optional helper method to retrieve protocol specific metadata
       // about a certain peer in the network. If an info retrieval function is set,
       // but returns nil, it is assumed that the protocol handshake is still running.
       PeerInfo func(id discover.NodeID) interface{}
}</pre>

3. The `eth.Ethereum` struct contains a [`ProtocolManager`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97), which include one `p2p.Protocol` for every supported protocol version; see [`eth/handler.go#L66-L97`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97): <pre>type ProtocolManager struct {
       networkID uint64
       fastSync  uint32 // Flag whether fast sync is enabled (gets disabled if we already have blocks)
       acceptTxs uint32 // Flag whether we're considered synchronised (enables transaction processing)
       txpool      txPool
       blockchain  *core.BlockChain
       chainconfig *params.ChainConfig
       maxPeers    int
       downloader *downloader.Downloader
       fetcher    *fetcher.Fetcher
       peers      *peerSet
       <b>SubProtocols []p2p.Protocol</b>
       eventMux      *event.TypeMux
       txsCh         chan core.NewTxsEvent
       txsSub        event.Subscription
       minedBlockSub *event.TypeMuxSubscription
       // channels for fetcher, syncer, txsyncLoop
       newPeerCh   chan *peer
       txsyncCh    chan *txsync
       quitSync    chan struct{}
       noMorePeers chan struct{}
       // wait group is used for graceful shutdowns during downloading
       // and processing
       wg sync.WaitGroup
}</pre>

4. `ProtocolManager.SubProtocols` is assigned a `p2p.Protocol` for every supported protocol; see [`eth/handler.go#L132`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L132):
<pre>// Initiate a sub-protocol for every implemented version we can handle
manager.SubProtocols = make([]p2p.Protocol, 0, len(ProtocolVersions))
   for i, version := range ProtocolVersions {
       // Skip protocol version if incompatible with the mode of operation
       if mode == downloader.FastSync && version < eth63 {
           continue
       }
       // Compatible; initialise the sub-protocol
       version := version // Closure for the run
       <b>manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol{
           Name:    ProtocolName,
           Version: version,
           Length:  ProtocolLengths[i],
           Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error {
               peer := manager.newPeer(int(version), p, rw)
               select {
               case manager.newPeerCh &lt;- peer:
                   manager.wg.Add(1)
                   defer manager.wg.Done()
                   return manager.handle(peer)</b>
               case &lt;-manager.quitSync:
                   return p2p.DiscQuitting
               }
           },
           NodeInfo: func() interface{} {
               return manager.NodeInfo()
           },
           PeerInfo: func(id discover.NodeID) interface{} {
               if p := manager.peers.Peer(fmt.Sprintf("%x", id[:8])); p != nil {
                   return p.Info()
               }
               return nil
           },
       })
   }</pre>

4. Each of the `SubProtocols` is defines a `Run` method that calls the `ProtocolManager`'s `handle()` method; see [`eth/handler.go#L142`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L142), contained in the preceding code snippet: <pre><b>Run</b>: func(p *p2p.Peer, rw p2p.MsgReadWriter) error {
    peer := manager.newPeer(int(version), p, rw)
    select {
    case manager.newPeerCh &lt;- peer:
        manager.wg.Add(1)
        defer manager.wg.Done()
        <b>return manager.handle(peer)</b>
    case &lt;-manager.quitSync:
        return p2p.DiscQuitting
    }
}</pre>

5. The `Run` method of each `SubProtocol` is called when `geth` starts the `Node`:

  | Line | Code |
  | --- |   --- |
  | [node/node.go#L138](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L138) | ``` func (n *Node) Start() error``` |
  | [node/node.go#L196](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L196) | ```if err := running.Start(); err != nil ``` |
  | [p2p/server.go#L504](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L504) | ```go srv.run(dialer)``` |
  | [p2p/server.go#L894](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L894) | ```remoteRequested, err := p.run()``` |
  | [p2p/peer.go#L197](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L197) | ```p.startProtocols(writeStart, writeErr)``` |
  | [p2p/peer.go#L348](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L348) | ```err := proto.Run(p, rw)``` |
   
6. An infinite loop handles incoming messages from the connected peer. See [eth/handler.go#L307-L313](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L307-L313):
  ```
	// main loop. handle incoming messages.
	for {
		if err := pm.handleMsg(p); err != nil {
			p.Log().Debug("Ethereum message handling failed", "err", err)
			return err
		}
	}
  ```

## Handling the message

1. `handleMsg()` reads the message from the peer; see [eth/handler.go#L320](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L320):
```msg, err := p.rw.ReadMsg()```

2. The message is of type `NewBlockMsg`, so the block data is decoded and scheduled for import:
[eth/handler.go#L634-L664](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L634-L664)
```case msg.Code == NewBlockMsg:```

3. The block fetcher then tries to import the new block; see
[eth/fetcher/fetcher.go#L313](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L313
```f.insert(op.origin, op.block)```

4. If the block header validates correctly it is propagated to the node's peers; see [eth/fetcher/fetcher.go#L654-L657](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L654-L657)
```go f.broadcastBlock(block, true)```

5. The block is processed; see [eth/fetcher/fetcher.go#L669-L672](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L669-L672)
```
if _, err := f.insertChain(types.Blocks{block}); err != nil {
    glog.V(logger.Warn).Infof("Peer %s: block #%d [%x…] import failed: %v", peer, block.NumberU64(), hash[:4], err)
    return
}
```
See [core/blockchain.go#L1147](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L1147)
```receipts, logs, usedGas, err := bc.processor.Process(block, state, bc.vmConfig)```

6. If the block is processed successfully it is committed to the database; see [core/blockchain.go#L902](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L902)
```root, err := state.Commit(bc.chainConfig.IsEIP158(block.Number()))```

7. The new block is written to the chain; see [core/blockchain.go#L900](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L900)
```rawdb.WriteBlock(batch, block)```
