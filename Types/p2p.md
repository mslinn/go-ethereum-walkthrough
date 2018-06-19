# Peer to Peer {#p2p}
These types define Ethereum clients, servers and peers.

## `Protocol` {#protocol}
A [`Protocol`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L25-L55) `struct` is created for every supported protocol when `geth` starts (the startup sequence is not shown here); see [`p2p/protocol.go#L26-L55`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L26-L55): 
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

## `ProtocolManager` {#protocol_manager}
A [`ProtocolManager`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97) includes one `Protocol` for every supported protocol version; see [`eth/handler.go#L66-L97`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97): 
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

_More to come..._