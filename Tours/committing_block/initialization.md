# Initialization for Committing a Block {#initialization}

## [Types](initializing_types.md#types) {#types}

## Walkthrough {#walkthrough}

1. `geth` sets up a [`Node`](/Types/committing_block//initializing_types.md#node); see [`cmd/geth/main.go#L236`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L236):
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

2. The `makeFullNode` method is shown below; see [`cmd/geth/config.go#L153-L179`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L153-L179). It invokes `makeConfigNode`, which returns a tuple consisting of the new `Node` and the corresponding [`eth.Config`](https://github.com/ethereum/go-ethereum/blob/master/eth/config.go#L76-L117) data); see `#2a` below. On the next line (see `#2b` below) `RegisterEthService` (shown [later](#RegisterEthService)) uses the tuple to add the new Ethereum client to the stack (TODO describe the `Node` stack); see [`cmd/geth/config.go#L156`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L156):
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

4. <a name="RegisterEthService"></a> `RegisterEthService` is a publicly visible function; see [`cmd/utils/flags.go#L1126-L1146`](https://github.com/ethereum/go-ethereum/blob/master/cmd/utils/flags.go#L1126-L1146). As previously mentioned, `RegisterEthService` uses a reference to a new `Node` (which is actually a new Ethereum client) and the corresponding `eth.Config` data to add the new Ethereum client. The `eth.New` method, shown in `#4a` and `#4b`, returns the registered Ethereum client. `TODO` the terminology is confusing; the terms "Ethereum client", "stack" and "Ethereum service" all seem to refer to the same object.
    ```go
    // RegisterEthService adds an Ethereum client to the stack.
    func RegisterEthService(stack *node.Node, cfg *eth.Config) {
        var err error
        if cfg.SyncMode == downloader.LightSync {
            err = stack.Register(func(ctx *node.ServiceContext) (node.Service, error) {
                return les.New(ctx, cfg) // <<=== #4a
            })
        } else {
            err = stack.Register(func(ctx *node.ServiceContext) (node.Service, error) {
                fullNode, err := eth.New(ctx, cfg)  // <<=== #4b
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

5. The `eth.New` method accepts a `node.ServiceContext` and a reference to an `eth.Config` instance and returns a reference to a new `LightEthereum` instance; see [`les/backend.go#L83-L140`](https://github.com/ethereum/go-ethereum/blob/master/les/backend.go#L83-L140). The new `LightEthereum` instance contains a [`ProtocolManager`](/Types/p2p.md#protocol_manager), which includes one `p2p.Protocol` for every supported protocol version; see #5. 
  ```go
  func New(ctx *node.ServiceContext, config *eth.Config) (*LightEthereum, error) {
    chainDb, err := eth.CreateDB(ctx, config, "lightchaindata")
    if err != nil {
        return nil, err
    }
    chainConfig, genesisHash, genesisErr := core.SetupGenesisBlock(chainDb, config.Genesis)
    if _, isCompat := genesisErr.(*params.ConfigCompatError); genesisErr != nil && !isCompat {
        return nil, genesisErr
    }
    log.Info("Initialised chain configuration", "config", chainConfig)

    peers := newPeerSet()
    quitSync := make(chan struct{})

    leth := &LightEthereum{
        config:           config,
        chainConfig:      chainConfig,
        chainDb:          chainDb,
        eventMux:         ctx.EventMux,
        peers:            peers,
        reqDist:          newRequestDistributor(peers, quitSync),
        accountManager:   ctx.AccountManager,
        engine:           eth.CreateConsensusEngine(ctx, &config.Ethash, chainConfig, chainDb),
        shutdownChan:     make(chan bool),
        networkId:        config.NetworkId,
        bloomRequests:    make(chan chan *bloombits.Retrieval),
        bloomIndexer:     eth.NewBloomIndexer(chainDb, light.BloomTrieFrequency),
        chtIndexer:       light.NewChtIndexer(chainDb, true),
        bloomTrieIndexer: light.NewBloomTrieIndexer(chainDb, true),
    }

    leth.relay = NewLesTxRelay(peers, leth.reqDist)
    leth.serverPool = newServerPool(chainDb, quitSync, &leth.wg)
    leth.retriever = newRetrieveManager(peers, leth.reqDist, leth.serverPool)
    leth.odr = NewLesOdr(chainDb, leth.chtIndexer, leth.bloomTrieIndexer, leth.bloomIndexer, leth.retriever)
    if leth.blockchain, err = light.NewLightChain(leth.odr, leth.chainConfig, leth.engine); err != nil {
        return nil, err
    }
    leth.bloomIndexer.Start(leth.blockchain)
    // Rewind the chain in case of an incompatible config upgrade.
    if compat, ok := genesisErr.(*params.ConfigCompatError); ok {
        log.Warn("Rewinding chain to upgrade configuration", "err", compat)
        leth.blockchain.SetHead(compat.RewindTo)
        rawdb.WriteChainConfig(chainDb, genesisHash, chainConfig)
    }

    leth.txPool = light.NewTxPool(leth.chainConfig, leth.blockchain, leth.relay)
    // <<=== #6 on next line
    if leth.protocolManager, err = NewProtocolManager(leth.chainConfig, true, ClientProtocolVersions, config.NetworkId, leth.eventMux, leth.engine, leth.peers, leth.blockchain, nil, chainDb, leth.odr, leth.relay, leth.serverPool, quitSync, &leth.wg); err != nil { 
        return nil, err
    }
    leth.ApiBackend = &LesApiBackend{leth, nil}
    gpoParams := config.GPO
    if gpoParams.Default == nil {
        gpoParams.Default = config.GasPrice
    }
    leth.ApiBackend.gpo = gasprice.NewOracle(leth.ApiBackend, gpoParams)
    return leth, nil
}
```
  
5. The Ethereum sub-protocol manages peers within the Ethereum network. The `NewProtocolManager` method returns a new Ethereum sub-protocol manager. `NewProtocolManager` is a long method, so only a portion of it is shown here; see [`eth/handler.go#L99-L182`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L99-L182) to see all of it. `Suggestion`: refactor `NewProtocolManager` into smaller methods. `ProtocolManager.SubProtocols` is assigned a `p2p.Protocol` for every supported protocol; see [`eth/handler.go#L132`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L132):
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
               Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error {          // <<=== #7b start
                   peer := manager.newPeer(int(version), p, rw)
                   select {
                   case manager.newPeerCh &lt;- peer:
                       manager.wg.Add(1)
                       defer manager.wg.Done()
                       return manager.handle(peer)</b>
                   case &lt;-manager.quitSync:
                       return p2p.DiscQuitting
                   }
               },                                                            // <<=== #7b end
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
  
  f. TODO write explanation for this step; see [`p2p/server.go#L894`](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L894) 
  ```go
  remoteRequested, err := p.run()
  ```
  
  g. TODO write explanation for this step;see [`p2p/peer.go#L197`](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L197) 
  ```go
  p.startProtocols(writeStart, writeErr)
  ```
  
  h. TODO write explanation for this step;see [`p2p/peer.go#L348`](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L348)
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
