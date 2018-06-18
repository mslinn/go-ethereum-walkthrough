# Following a Block From the Socket Until Committed to the Blockchain

_This is a modified version of [go-ethereum-code-walkthrough.md](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [gsalgado](https://github.com/gsalgado)._

Note: All links to the code are based on `master` as it was when this document was last modified. Line numbers might shift as a result.

## Preconditons
* `geth` is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## Initialization

1. `geth` sets up a full `Node`; see [`cmd/geth/main.go#L236`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L236):
```node := makeFullNode(ctx)```

2. `geth` registers an instance of the `eth.Ethereum` service with that `Node`; see [`cmd/geth/config.go#L156`](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/config.go#L156):
```utils.RegisterEthService(stack, &cfg.Eth)```

3. The `eth.Ethereum` service contains a `ProtocolManager`, which includes one `SubProtocol` for every supported protocol version; see [`eth/handler.go#L132`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L132):
```manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol ...```

4. Each of the `SubProtocols` is defines a `Run` method that calls the `ProtocolManager`'s `handle()` method; see [`eth/handler.go#L142`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L142):
```return manager.handle(peer)```

5. The `Run` method of each `SubProtocol` is called when `geth` starts the `Node`:
[cmd/geth/main.go#L188](https://github.com/ethereum/go-ethereum/blob/master/cmd/geth/main.go#L188)
[node/node.go#L142](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L142)
[node/node.go#L206](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L206)
[p2p/server.go#L406](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L406)
[p2p/server.go#L542](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L542)
[p2p/server.go#L742](https://github.com/ethereum/go-ethereum/blob/master/p2p/server.go#L742)
[p2p/peer.go#L150](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L150)
[p2p/peer.go#L303](https://github.com/ethereum/go-ethereum/blob/master/p2p/peer.go#L303)

which loops infinitely, handling incoming messages from the connected peer:
[eth/handler.go#L311](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L311)


## Handling the message

handleMsg() reads the msg from the peer:
[eth/handler.go#L324](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L324)

which in our case is a NewBlockMsg, so the block data is decoded and scheduled for import:
[eth/handler.go#L629](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L629)

The block fetcher will then try to import the new block:
[eth/fetcher/fetcher.go#L313](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L313)

If the block header validates correctly it is propagated to the node's peers:
[eth/fetcher/fetcher.go#L672](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L672)

processed:
[eth/fetcher/fetcher.go#L684](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L684)
[core/blockchain.go#L959](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L959)

and if that results in a valid state, it is committed to the database:
[core/blockchain.go#L971](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L971)

and written to the chain:
[core/blockchain.go#L984](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L984)