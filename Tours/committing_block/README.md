# Tour: Committing a Block {#committing-block}

_Following a block from the socket until it is committed to the blockchain._

All links to the code are based on the `master` branch as it was when this document was last modified. Line numbers might shift as a result.

## Attribution
This is a modified version of [go-ethereum-code-walkthrough.md](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [gsalgado](https://github.com/gsalgado). Code references were updated, more detail was added, links to types were provided, a redundant step was removed, spelling was corrected and code snippets were added.

## Preconditions
* `geth` is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected to a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## Initialization

1. `geth` sets up a full [`Node`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L40-L74); see [`cmd/geth/main.go#L236`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L236):
  ```go
// geth is the main entry point into the system if no special subcommand is ran.
// It creates a default node based on the command line arguments and runs it in
// blocking mode, waiting for it to be shut down.
func geth(ctx *cli.Context) error {
        node := makeFullNode(ctx)
        startNode(ctx, node)   // <<=== #1
        node.Wait()
        return nil
}```

2. `geth` registers an instance of the [`eth.Ethereum`](https://github.com/ethereum/go-ethereum/blob/master/eth/config.go#L76-L117) service with that `Node`; see [`cmd/geth/config.go#L156`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L156):
  ```go
  func makeFullNode(ctx *cli.Context) *node.Node {
        stack, cfg := makeConfigNode(ctx)
        utils.RegisterEthService(stack, &cfg.Eth)   // <<=== #2
        if ctx.GlobalBool(utils.DashboardEnabledFlag.Name) {
            utils.RegisterDashboardService(stack, &cfg.Dashboard, gitCommit)
  }
  ```

3. A [`Protocol`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L25-L55) `struct` is created for every supported protocol when `geth` starts (the startup sequence is not shown here); see [`p2p/protocol.go#L26-L55`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L26-L55): 
  ```go
  // Protocol represents a P2P subprotocol implementation.
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
}
```

4. The `eth.Ethereum` struct contains a [`ProtocolManager`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97), which include one `p2p.Protocol` for every supported protocol version; see [`eth/handler.go#L66-L97`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97): 
  ```go
  type ProtocolManager struct {
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
       
       SubProtocols []p2p.Protocol     // <<=== #4
       
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
}
```

5. `ProtocolManager.SubProtocols` is assigned a `p2p.Protocol` for every supported protocol; see [`eth/handler.go#L132`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L132):
```go
// Initiate a sub-protocol for every implemented version we can handle
manager.SubProtocols = make([]p2p.Protocol, 0, len(ProtocolVersions))
   for i, version := range ProtocolVersions {
       // Skip protocol version if incompatible with the mode of operation
       if mode == downloader.FastSync && version < eth63 {
           continue
       }
       // Compatible; initialise the sub-protocol
       version := version // Closure for the run
       manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol{ // <<=== #5
           Name:    ProtocolName,
           Version: version,
           Length:  ProtocolLengths[i],
           Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error {          // <<=== #6 start
               peer := manager.newPeer(int(version), p, rw)
               select {
               case manager.newPeerCh &lt;- peer:
                   manager.wg.Add(1)
                   defer manager.wg.Done()
                   return manager.handle(peer)</b>
               case &lt;-manager.quitSync:
                   return p2p.DiscQuitting
               }
           },                                                            // <<=== #6 end
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
   }
```

6. Each of the `SubProtocols` defines a `Run` method that calls the `ProtocolManager`'s `handle()` method; see [`eth/handler.go#L142`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L142), contained in the preceding code snippet.

7. The `Run` method of each `SubProtocol` is called when `geth` starts the `Node`. 
  
  a. First the `Node.Start()` method is invoked; see [`node/node.go#L138-L228`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L138-L228) 
  ```go
  // Start create a live P2P node and starts running it.
  func (n *Node) Start() error {
  ```
  
  b. Peer to peer networking (for Ethereum clients) is described in the [go-ethereum documentation](https://github.com/ethereum/go-ethereum/wiki/Peer-to-Peer). The `Server` type is oddly enough used to manage Ethereum clients, and is defined in [`node/server.go#147-178`](https://github.com/ethereum/go-ethereum/blob/master/node/server.go#L147-178) like this:
  ```go
  type Server struct {
       // Config fields may not be modified while the server is running.
       Config

        // Hooks for testing. These are useful because we can inhibit
        // the whole protocol stack.
        newTransport func(net.Conn) transport
        newPeerHook  func(*Peer)

        lock    sync.Mutex // protects running
        running bool

        ntab         discoverTable
        listener     net.Listener
        ourHandshake *protoHandshake
        lastLookup   time.Time
        DiscV5       *discv5.Network

        // These are for Peers, PeerCount (and nothing else).
        peerOp     chan peerOpFunc
        peerOpDone chan struct{}

        quit          chan struct{}
        addstatic     chan *discover.Node
        removestatic  chan *discover.Node
        posthandshake chan *conn
        addpeer       chan *conn
        delpeer       chan peerDrop
        loopWG        sync.WaitGroup // loop, listenLoop
        peerFeed      event.Feed
        log           log.Logger
  }
  ```
  
  c. Within `Node.Start()`, [this line](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L165) defines a variable called `running` that will point to a new `Server` instance:
  ```go
  running := &p2p.Server{Config: n.serverConfig}
  ```
  
  d. The supported protocols are appended and the p2p server is started; see [`node/node.go#L196-L198`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L196-L198)
  ```go
  // Gather the protocols and start the freshly assembled P2P server
  for _, service := range services {
      running.Protocols = append(running.Protocols, service.Protocols()...)
  }
  if err := running.Start(); err != nil {
      return convertFileLockError(err)
  }	
```
  
  e. Near the end of `Node.Start()`, we see [`p2p/server.go#L504`](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L504)
  ```go
go srv.run(dialer)
```
  
  f. see [`p2p/server.go#L894`](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L894) 
  ```go
remoteRequested, err := p.run()
```
  
  g. see [`p2p/peer.go#L197`](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L197) 
  ```go
p.startProtocols(writeStart, writeErr)
```
  
  h. see [`p2p/peer.go#L348`](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L348)
  ```go
err := proto.Run(p, rw)
```
   
6. An infinite loop handles incoming messages from the connected peer. See [eth/handler.go#L307-L313](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L307-L313):
  ```go
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
```go 
msg, err := p.rw.ReadMsg()
```

2. The message is of type `NewBlockMsg`, so the block data is decoded and scheduled for import:
[eth/handler.go#L634-L664](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L634-L664)
```go
case msg.Code == NewBlockMsg:
```

3. The block fetcher then tries to import the new block; see
[eth/fetcher/fetcher.go#L313](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L313
```go
f.insert(op.origin, op.block)
```

4. If the block header validates correctly it is propagated to the node's peers; see [eth/fetcher/fetcher.go#L654-L657](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L654-L657)
```go 
go f.broadcastBlock(block, true)
```

5. The block is processed; see [eth/fetcher/fetcher.go#L669-L672](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L669-L672)
```go
if _, err := f.insertChain(types.Blocks{block}); err != nil {
      glog.V(logger.Warn).Infof("Peer %s: block #%d [%x…] import failed: %v", peer, block.NumberU64(), hash[:4], err)
      return
}
```
See [core/blockchain.go#L1147](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L1147)
```go
receipts, logs, usedGas, err := bc.processor.Process(block, state, bc.vmConfig)
```

6. If the block is processed successfully it is committed to the database; see [core/blockchain.go#L902](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L902)
```go
root, err := state.Commit(bc.chainConfig.IsEIP158(block.Number()))
```

7. The new block is written to the chain; see [core/blockchain.go#L900](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L900)
```go
rawdb.WriteBlock(batch, block)
```
