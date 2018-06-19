# Peer to Peer {#p2p}
These types define Ethereum clients, servers and peers.

## `Config` {#config}
[This data structure](https://github.com/ethereum/go-ethereum/blob/master/node/config.go#L49-L150) configures an Ethereum [`Node`](#node).

```go
type Config struct {
    // Name sets the instance name of the node. It must not contain the / character and is
    // used in the devp2p node identifier. The instance name of geth is "geth". If no
    // value is specified, the basename of the current executable is used.
    Name string `toml:"-"`

    // UserIdent, if set, is used as an additional component in the devp2p node identifier.
    UserIdent string `toml:",omitempty"`

    // Version should be set to the version number of the program. It is used
    // in the devp2p node identifier.
    Version string `toml:"-"`

    // DataDir is the file system folder the node should use for any data storage
    // requirements. The configured data directory will not be directly shared with
    // registered services, instead those can use utility methods to create/access
    // databases or flat files. This enables ephemeral nodes which can fully reside
    // in memory.
    DataDir string

    // Configuration of peer-to-peer networking.
    P2P p2p.Config

    // KeyStoreDir is the file system folder that contains private keys. The directory can
    // be specified as a relative path, in which case it is resolved relative to the
    // current directory.
    //
    // If KeyStoreDir is empty, the default location is the "keystore" subdirectory of
    // DataDir. If DataDir is unspecified and KeyStoreDir is empty, an ephemeral directory
    // is created by New and destroyed when the node is stopped.
    KeyStoreDir string `toml:",omitempty"`

    // UseLightweightKDF lowers the memory and CPU requirements of the key store
    // scrypt KDF at the expense of security.
    UseLightweightKDF bool `toml:",omitempty"`

    // NoUSB disables hardware wallet monitoring and connectivity.
    NoUSB bool `toml:",omitempty"`

    // IPCPath is the requested location to place the IPC endpoint. If the path is
    // a simple file name, it is placed inside the data directory (or on the root
    // pipe path on Windows), whereas if it's a resolvable path name (absolute or
    // relative), then that specific path is enforced. An empty path disables IPC.
    IPCPath string `toml:",omitempty"`

    // HTTPHost is the host interface on which to start the HTTP RPC server. If this
    // field is empty, no HTTP API endpoint will be started.
    HTTPHost string `toml:",omitempty"`

    // HTTPPort is the TCP port number on which to start the HTTP RPC server. The
    // default zero value is/ valid and will pick a port number randomly (useful
    // for ephemeral nodes).
    HTTPPort int `toml:",omitempty"`

    // HTTPCors is the Cross-Origin Resource Sharing header to send to requesting
    // clients. Please be aware that CORS is a browser enforced security, it's fully
    // useless for custom HTTP clients.
    HTTPCors []string `toml:",omitempty"`

    // HTTPVirtualHosts is the list of virtual hostnames which are allowed on incoming requests.
    // This is by default {'localhost'}. Using this prevents attacks like
    // DNS rebinding, which bypasses SOP by simply masquerading as being within the same
    // origin. These attacks do not utilize CORS, since they are not cross-domain.
    // By explicitly checking the Host-header, the server will not allow requests
    // made against the server with a malicious host domain.
    // Requests using ip address directly are not affected
    HTTPVirtualHosts []string `toml:",omitempty"`

    // HTTPModules is a list of API modules to expose via the HTTP RPC interface.
    // If the module list is empty, all RPC API endpoints designated public will be
    // exposed.
    HTTPModules []string `toml:",omitempty"`

    // WSHost is the host interface on which to start the websocket RPC server. If
    // this field is empty, no websocket API endpoint will be started.
    WSHost string `toml:",omitempty"`

    // WSPort is the TCP port number on which to start the websocket RPC server. The
    // default zero value is/ valid and will pick a port number randomly (useful for
    // ephemeral nodes).
    WSPort int `toml:",omitempty"`

    // WSOrigins is the list of domain to accept websocket requests from. Please be
    // aware that the server can only act upon the HTTP request the client sends and
    // cannot verify the validity of the request header.
    WSOrigins []string `toml:",omitempty"`

    // WSModules is a list of API modules to expose via the websocket RPC interface.
    // If the module list is empty, all RPC API endpoints designated public will be
    // exposed.
    WSModules []string `toml:",omitempty"`

    // WSExposeAll exposes all API modules via the WebSocket RPC interface rather
    // than just the public ones.
    //
    // *WARNING* Only set this if the node is running in a trusted network, exposing
    // private APIs to untrusted users is a major security risk.
    WSExposeAll bool `toml:",omitempty"`

    // Logger is a custom logger to use with the p2p.Server.
    Logger log.Logger `toml:",omitempty"`
}
```

## `Node` {#node}
An [entire package](https://godoc.org/github.com/ethereum/go-ethereum/node) defines the behavior of a [`Node`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L40-L74).

```go
// Node is a container on which services can be registered.
type Node struct {
	eventmux *event.TypeMux // Event multiplexer used between the services of a stack
	config   *Config
	accman   *accounts.Manager

	ephemeralKeystore string         // if non-empty, the key directory that will be removed by Stop
	instanceDirLock   flock.Releaser // prevents concurrent use of instance directory

	serverConfig p2p.Config
	server       *p2p.Server // Currently running P2P networking layer

	serviceFuncs []ServiceConstructor     // Service constructors (in dependency order)
	services     map[reflect.Type]Service // Currently running services

	rpcAPIs       []rpc.API   // List of APIs currently provided by the node
	inprocHandler *rpc.Server // In-process RPC request handler to process the API requests

	ipcEndpoint string       // IPC endpoint to listen at (empty = IPC disabled)
	ipcListener net.Listener // IPC RPC listener socket to serve API requests
	ipcHandler  *rpc.Server  // IPC RPC request handler to process the API requests

	httpEndpoint  string       // HTTP endpoint (interface + port) to listen at (empty = HTTP disabled)
	httpWhitelist []string     // HTTP RPC modules to allow through this endpoint
	httpListener  net.Listener // HTTP RPC listener socket to server API requests
	httpHandler   *rpc.Server  // HTTP RPC request handler to process the API requests

	wsEndpoint string       // Websocket endpoint (interface + port) to listen at (empty = websocket disabled)
	wsListener net.Listener // Websocket RPC listener socket to server API requests
	wsHandler  *rpc.Server  // Websocket RPC request handler to process the API requests

	stop chan struct{} // Channel to wait for termination notifications
	lock sync.RWMutex

	log log.Logger
}
```

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

## `Server` {#server}
Peer to peer networking (for Ethereum clients) is described in the [go-ethereum documentation](https://github.com/ethereum/go-ethereum/wiki/Peer-to-Peer). The `Server` type is oddly enough used to manage Ethereum clients, and is defined in [`node/server.go#147-178`](https://github.com/ethereum/go-ethereum/blob/master/node/server.go#L147-178) like this:

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

_More to come..._