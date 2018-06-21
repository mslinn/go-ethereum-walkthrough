# Tour: Committing a Block {#committing-block}

_This tour follows a block in the `geth` client from the socket until it is committed to the blockchain._

All links to the code are based on the `master` branch as it was when this document was last modified. Line numbers might shift as a result.

## Types {#types}
The following `go-ethereum` types are referenced in this tour:
 * [Common](/Types/common.md)
 * [Database](/Types/database.md)
 * [Peer to peer](/Types/p2p.md)

The following types are used in the initialization phase of this tour:

### `cli.Context` {#Context}
This publicly visible type is provided by the [`github.com/urfave/cli.v1`](hhttps://github.com/urfave/cli#cli) dependency:
> "cli is a simple, fast, and fun package for building command line apps in Go. The goal is to enable developers to write fast and distributable command line applications in an expressive way.

```go
// Context is a type that is passed through to
// each Handler action in a cli application. Context
// can be used to retrieve context-specific Args and
// parsed command-line options.
type Context struct {
    App *App
    Command Command
    shellComplete bool
    flagSet *flag.FlagSet
    setFlags map[string]bool
    parentContext *Context
}
```

### `gethConfig` {#gethConfig}
The `gethConfig` private type is only visible within the `main` package for the `geth` command. The package `main` is the top-level package for Go programs, and tells the Go compiler that the package should compile as an executable program instead of a shared library.

```go
type gethConfig struct {
    Eth eth.Config
    Shh whisper.Config
    Node node.Config
    Ethstats ethstatsConfig
    Dashboard dashboard.Config
}
```

## Attribution {#attribution}
Following is a modified version of [go-ethereum-code-walkthrough.md](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [gsalgado](https://github.com/gsalgado). Code references were updated, more detail was added, links to types were provided, a redundant step was removed, spelling was corrected and code snippets were added.

## Preconditions {#preconditions}
* Program is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected to a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## Initialization {#initialization}
_This section is messed up right now due to reorganization, check back in a bit._

1. `geth` sets up a full [`Node`](/Types/p2p.md#node); see [`cmd/geth/main.go#L236`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L236):
  ```go
    // geth is the main entry point into the system if no special subcommand is ran.
    // It creates a default node based on the command line arguments and runs it in
    // blocking mode, waiting for it to be shut down.
    func geth(ctx *cli.Context) error {
            node := makeFullNode(ctx)    // <<=== #1
            startNode(ctx, node)
            node.Wait()
            return nil
    }
  ```

2. The `makeFullNode` method is shown below; see [`cmd/geth/config.go#L153-L179`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L153-L179). It invokes `makeConfigNode`, which returns a tuple consisting of the new `Node` and the corresponding [`eth.Config`](https://github.com/ethereum/go-ethereum/blob/master/eth/config.go#L76-L117) data); see `#2a` below. On the next line (see `#2b` below) `RegisterEthService` (shown [later](#RegisterEthService)) uses the tuple to add the new Ethereum client to the stack; see [`cmd/geth/config.go#L156`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L156):
    ```go
    func makeFullNode(ctx *cli.Context) *node.Node {
           stack, cfg := makeConfigNode(ctx)          // <<=== #2a
           utils.RegisterEthService(stack, &cfg.Eth)  // <<=== #2b
           if ctx.GlobalBool(utils.DashboardEnabledFlag.Name) {
           utils.RegisterDashboardService(stack, &cfg.Dashboard, gitCommit)
    }
    ```

3. The private `makeConfigNode` method added to the `cli.Context` type in `cmd/geth/config.go` uses the user-supplied command line to return a tuple containing a reference to a new `Node` and the new instance of `gethConfig` used to construct the `Node`; see [`cmd/geth/config.go#L110-L141`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L110-L141). It starts by creating an instance of [`gethConfig`](#gethConfig), described above; the two properties of interest are `Eth` and `Node`. The former property is assigned a default value (`eth.DefaultConfig`, see `#3a`, and shown in detail next), while the latter property's value is computed by the `defaultNodeConfig` method, see `#3b` and shown in detail following.

  ```go
    func makeConfigNode(ctx *cli.Context) (*node.Node, gethConfig) {
        // Load defaults.
        cfg := gethConfig{
            Eth:       eth.DefaultConfig,      // <<=== #3a
            Shh:       whisper.DefaultConfig,
            Node:      defaultNodeConfig(),    // <<=== #3b
            Dashboard: dashboard.DefaultConfig,
        }
    
        // Load config file.
        if file := ctx.GlobalString(configFileFlag.Name); file != "" {
            if err := loadConfig(file, &cfg); err != nil {
                utils.Fatalf("%v", err)
            }
        }
    
        // Apply flags.
        utils.SetNodeConfig(ctx, &cfg.Node)
        stack, err := node.New(&cfg.Node)
        if err != nil {
            utils.Fatalf("Failed to create the protocol stack: %v", err)
        }
        utils.SetEthConfig(ctx, stack, &cfg.Eth)
        if ctx.GlobalIsSet(utils.EthStatsURLFlag.Name) {
            cfg.Ethstats.URL = ctx.GlobalString(utils.EthStatsURLFlag.Name)
        }
    
        utils.SetShhConfig(ctx, stack, &cfg.Shh)
        utils.SetDashboardConfig(ctx, &cfg.Dashboard)
    
        return stack, cfg
    }
    
    ```

  a. `eth.DefaultConfig` looks like this; see [`eth/config.go#L36-L58`](https://github.com/ethereum/go-ethereum/blob/master/eth/config.go#L36-L58):
    ```go
    // DefaultConfig contains default settings for use on the Ethereum main net.
    var DefaultConfig = Config{
        SyncMode: downloader.FastSync,
        Ethash: ethash.Config{
            CacheDir:       "ethash",
            CachesInMem:    2,
            CachesOnDisk:   3,
            DatasetsInMem:  1,
            DatasetsOnDisk: 2,
        },
        NetworkId:     1,
        LightPeers:    100,
        DatabaseCache: 768,
        TrieCache:     256,
        TrieTimeout:   60 * time.Minute,
        GasPrice:      big.NewInt(18 * params.Shannon),
    
        TxPool: core.DefaultTxPoolConfig,
        GPO: gasprice.Config{
            Blocks:     20,
            Percentile: 60,
        },
    }
    ```

  b. The `defaultNodeConfig` method returns a `node.Config` instance based on `node.DefaultConfig`, see [`cmd/geth/config.go#L100-L108`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L100-L108):
  ```go
    func defaultNodeConfig() node.Config {
        cfg := node.DefaultConfig   // <<=== #3c
        cfg.Name = clientIdentifier
        cfg.Version = params.VersionWithCommit(gitCommit)
        cfg.HTTPModules = append(cfg.HTTPModules, "eth", "shh")
        cfg.WSModules = append(cfg.WSModules, "eth", "shh")
        cfg.IPCPath = "geth.ipc"
        return cfg
    }
  ```
  
  c. `node.DefaultConfig` looks like this; see [`node/defaults.go#L36-L49`](https://github.com/ethereum/go-ethereum/blob/master/node/defaults.go#L36-L49):
  
  ```go
    // DefaultConfig contains reasonable default settings.
    var DefaultConfig = Config{
        DataDir: DefaultDataDir(),
        HTTPPort: DefaultHTTPPort,
        HTTPModules: []string{"net", "web3"},
        HTTPVirtualHosts: []string{"localhost"},
        WSPort: DefaultWSPort,
        WSModules: []string{"net", "web3"},
        P2P: p2p.Config{
            ListenAddr: ":30303",
            MaxPeers: 25,
            NAT: nat.Any(),
        },
    }
  ```

4. <a name="RegisterEthService"></a>`RegisterEthService` is a publicly visible function that resides in `cmd/utils/flags.go`; see [`cmd/utils/flags.go#L1126-L1146`](https://github.com/ethereum/go-ethereum/blob/master/cmd/utils/flags.go#L1126-L1146):
    ```go
    // RegisterEthService adds an Ethereum client to the stack.
    func RegisterEthService(stack *node.Node, cfg *eth.Config) {
        var err error
        if cfg.SyncMode == downloader.LightSync {
            err = stack.Register(func(ctx *node.ServiceContext) (node.Service, error) {
                return les.New(ctx, cfg)
            })
        } else {
            err = stack.Register(func(ctx *node.ServiceContext) (node.Service, error) {
                fullNode, err := eth.New(ctx, cfg)
                if fullNode != nil && cfg.LightServ > 0 {
                    ls, _ := les.NewLesServer(fullNode, cfg)
                    fullNode.AddLesServer(ls)
                }
                return fullNode, err
            })
        }
        if err != nil {
            Fatalf("Failed to register the Ethereum service: %v", err)
        }
    }
   ```

5. The Ethereum sub-protocol manages peers within the Ethereum network. The `NewProtocolManager` method in `eth/handler.go` returns a new Ethereum sub-protocol manager. The `eth.Ethereum` struct contains a [`ProtocolManager`](/Types/p2p.md#protocol_manager), which includes one `p2p.Protocol` for every supported protocol version. `NewProtocolManager` is a long method, so only a portion of it is shown here; see [`eth/handler.go#L99-L182`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L99-L182) to see all of it. `ProtocolManager.SubProtocols` is assigned a `p2p.Protocol` for every supported protocol; see [`eth/handler.go#L132`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L132):
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
           manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol{ // <<=== #6a
               Name:    ProtocolName,
               Version: version,
               Length:  ProtocolLengths[i],
               Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error {          // <<=== #6b start
                   peer := manager.newPeer(int(version), p, rw)
                   select {
                   case manager.newPeerCh &lt;- peer:
                       manager.wg.Add(1)
                       defer manager.wg.Done()
                       return manager.handle(peer)</b>
                   case &lt;-manager.quitSync:
                       return p2p.DiscQuitting
                   }
               },                                                            // <<=== #6b end
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

11. The `Run` method of each `SubProtocol` is called when `geth` starts the `Node`. 
  
  a. First the `Node.Start()` method is invoked; see [`node/node.go#L138-L228`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L138-L228) 
  ```go
  // Start create a live P2P node and starts running it.
  func (n *Node) Start() error {
  ```
   
  b. Within `Node.Start()`, [this line](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L165) defines a variable called `running` that will point to a new [`Server`](/Types/p2p.md#server) instance:
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
   
7. An infinite loop handles incoming messages from the connected peer. See [eth/handler.go#L307-L313](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L307-L313):
  ```go
    // main loop. handle incoming messages.
    for {
          if err := pm.handleMsg(p); err != nil {
              p.Log().Debug("Ethereum message handling failed", "err", err)
              return err
          }
    }
  ```

## Message Handling {#handling}
_This section is messed up right now due to reorganization, check back in a bit._

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

_More to come..._