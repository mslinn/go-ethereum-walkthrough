# Peer to Peer {#p2p}
These types define Ethereum clients, servers and peers.

FYI, a Gitter channel exists for this topic: [`ethereum/devp2p`](https://gitter.im/ethereum/devp2p).

## `LightEthereum` {#LightEthereum}
The [Light Ethereum Protocol](https://github.com/ethereum/wiki/wiki/Light-client-protocol) is implemented in the `les` package. The Gitter channel is [`ethereum/light-client`](https://gitter.im/ethereum/light-client). From the [Parity documentation](https://wiki.parity.io/Light-Ethereum-Subprotocol-\(LES\)):

> The Light Ethereum Subprotocol (LES) is the protocol used by “light” clients, which only download block headers as they appear and fetch other parts of the blockchain on-demand. They provide full functionality in terms of safely accessing the blockchain, but do not mine and therefore do not take part in the consensus process. Full and archive nodes can also support the LES protocol besides ETH in order to be able to serve light nodes. It has been decided to create a separate sub-protocol in order to avoid interference with the consensus-critical ETH network and make it easier to update during the development phase. Some of the LES protocol messages are similar to the “new sync model” (ETH62/63) of the Ethereum Wire Protocol, with the addition of a few fields.

Here are some outtakes from the [original LES documentation](https://blog.ethereum.org/2017/01/07/introduction-light-client-dapp-developers/):

> Light clients do not receive pending transactions from the main Ethereum network. The only pending transactions a light client knows about are the ones that have been created and sent from that client. When a light client sends a transaction, it starts downloading entire blocks until it finds the sent transaction in one of the blocks, then removes it from the pending transaction set.

> Latency is the key performance parameter of a light client. It is usually in the 100-200ms order of magnitude, and it applies to every state/contract storage read, block and receipt set retrieval.

> ... at the moment you should not search for anything in the entire history because it will take an extremely long time.

> ... With garbage collection enabled, the database will function more like a cache, and a light client will be able to run with as low as 10Mb of storage space. Note that the current Geth implementation uses around 200Mb of memory, which can probably be further reduced. Bandwidth requirements are also lower when the client is not used heavily. Bandwidth used is usually well under 1Mb/hour when running idle, with an additional 2-3kb for an average state/storage request.



A [`LightEthereum`](https://github.com/ethereum/go-ethereum/blob/master/les/backend.go#L49-L81) is a secondary node on the Ethereum blockchain. These nodes differ from full `Node`s by not requiring as many resources, because they are not fully capable.

```go
type LightEthereum struct {
    config *eth.Config

    odr         *LesOdr
    relay       *LesTxRelay
    chainConfig *params.ChainConfig
    // Channel for shutting down the service
    shutdownChan chan bool
    // Handlers
    peers           *peerSet
    txPool          *light.TxPool
    blockchain      *light.LightChain
    protocolManager *ProtocolManager
    serverPool      *serverPool
    reqDist         *requestDistributor
    retriever       *retrieveManager
    // DB interfaces
    chainDb ethdb.Database // Block chain database

    bloomRequests   chan chan *bloombits.Retrieval // Channel receiving bloom data retrieval requests
    bloomIndexer, chtIndexer, bloomTrieIndexer *core.ChainIndexer

    ApiBackend *LesApiBackend

    eventMux       *event.TypeMux
    engine         consensus.Engine
    accountManager *accounts.Manager

    networkId     uint64
    netRPCService *ethapi.PublicNetAPI

    wg sync.WaitGroup
}
```

## `Protocol` {#protocol}

A [`Protocol`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L25-L55) `struct` is created for every supported protocol when `geth` starts. All members are public. See [`p2p/protocol.go#L26-L55`](https://github.com/ethereum/go-ethereum/blob/master/p2p/protocol.go#L26-L55): 
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
A [`ProtocolManager`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97) includes one `Protocol` for every supported protocol version. All members are private. See [`eth/handler.go#L66-L97`](https://github.com/ethereum/go-ethereum/blob/master/eth/handler.go#L66-L97): 
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

## `Server` {#server}
Peer to peer networking (for Ethereum clients) is described in the [go-ethereum documentation](https://github.com/ethereum/go-ethereum/wiki/Peer-to-Peer). The `Server` type is oddly enough used to manage Ethereum clients, and is defined in [`node/server.go#147-178`](https://github.com/ethereum/go-ethereum/blob/master/node/server.go#L147-178) like this; all members are private:

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
