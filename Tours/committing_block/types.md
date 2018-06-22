# Types for Committing a Block {#types}
The following `go-ethereum` types are referenced in this tour:
 * [Common](/Types/common.md)
 * [Database](/Types/database.md)
 * [Peer to peer](/Types/p2p.md)

The following types are used in the initialization phase of this tour:

## `cli.Context` {#Context}
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

## `gethConfig` {#gethConfig}
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

## `Node` {#node}
The [`Node`](https://github.com/ethereum/go-ethereum/blob/master/node/node.go#L40-L74) type is only used by `geth`.

An [entire package](https://godoc.org/github.com/ethereum/go-ethereum/node) defines the behavior of the `Node` type. The definition of `Node` contains many properties, none of which are exported; this means that `Node` state is only accessed from other packages via methods:

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

## `Service` {#service}
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