# Types for Committing a Block / Initialization {#types}
The following `go-ethereum` types are referenced in this phase of the **Committing a Block** tour:

 * [Common](/Types/common.md)
 * [Database](/Types/database.md)
 * [Peer to peer](/Types/p2p.md)

## Initialization Types {#init_types}
The following types are only used in the initialization phase of this tour:

## `Config` {#config}
[This data structure](https://github.com/ethereum/go-ethereum/blob/master/node/config.go#L49-L150) configures an Ethereum [`Node`](#node). All members are public.

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

### `Node` {#node}
`Node` type is only used by `geth`; see [`node/node.go#L39-L74`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L39-L74).

An [entire package](https://godoc.org/github.com/ethereum/go-ethereum/node) defines the behavior of the `Node` type. The definition of `Node` contains many properties, none of which are exported; this means that `Node` state is only accessed from other packages via methods.

`TODO` `Node` instances are often named `stack`; why is that? Seems important.

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

### `Service` {#service}
See [`node/service.go#L74-L98`](https://github.com/ethereum/go-ethereum/blob/master/node/service.go#L74-L98)

```go
// Service is an individual protocol that can be registered into a node.
//
// Notes:
//
// • Service life-cycle management is delegated to the node. The service is allowed to
// initialize itself upon creation, but no goroutines should be spun up outside of the
// Start method.
//
// • Restart logic is not required as the node will create a fresh instance
// every time a service is started.
type Service interface {
	// Protocols retrieves the P2P protocols the service wishes to start.
	Protocols() []p2p.Protocol

	// APIs retrieves the list of RPC descriptors the service provides
	APIs() []rpc.API

	// Start is called after all services have been constructed and the networking
	// layer was also initialized to spawn any goroutines required by the service.
	Start(server *p2p.Server) error

	// Stop terminates all goroutines belonging to the service, blocking until they
	// are all terminated.
	Stop() error
}
```

## Message Handling Types {#handling_types}

### `peer` {#peer}
This private type has only private members; see [`eth/peer.go#L75-L94`](https://github.com/ethereum/go-ethereum/blob/master/eth/peer.go#L75-L94).

```go
type peer struct {
    id string

    *p2p.Peer
    rw p2p.MsgReadWriter

    version  int         // Protocol version negotiated
    forkDrop *time.Timer // Timed connection dropper if forks aren't validated in time

    head common.Hash
    td   *big.Int
    lock sync.RWMutex

    knownTxs    *set.Set                  // Set of transaction hashes known to be known by this peer
    knownBlocks *set.Set                  // Set of block hashes known to be known by this peer
    queuedTxs   chan []*types.Transaction // Queue of transactions to broadcast to the peer
    queuedProps chan *propEvent           // Queue of blocks to broadcast to the peer
    queuedAnns  chan *types.Block         // Queue of blocks to announce to the peer
    term        chan struct{}             // Termination channel to stop the broadcaster
}
```
