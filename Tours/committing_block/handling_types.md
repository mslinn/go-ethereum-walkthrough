# Committing a Block / Message Handling Types {#types}
The following `go-ethereum` types are referenced in this phase of the **Committing a Block** tour:

 * [Common](/Types/common.md)
 * [Database](/Types/database.md)
 * [Peer to peer](/Types/p2p.md)

## Message Handling Types {#init_types}
The following types are only used in the Message Handling phase of this tour:

### `newBlockData`
See [`eth/protocol.go#L170-L174`](https://github.com/ethereum/go-ethereum/blob/master/eth/protocol.go#L170-L174).
```go
// newBlockData is the network packet for the block propagation message.
type newBlockData struct {
	Block *types.Block
	TD    *big.Int
}
```

### `Block `
See [`core/types.go#L144-L162`](https://github.com/ethereum/go-ethereum/blob/master/core/types/block.go#L144-L162).

```go
// Block represents an entire block in the Ethereum blockchain.
type Block struct {
header *Header
uncles []*Header
transactions Transactions

// caches
hash atomic.Value
size atomic.Value

// Td is used by package core to store the total difficulty
// of the chain up to and including the block.
td *big.Int

// These fields are used by package eth to track
// inter-peer block relay.
ReceivedAt time.Time
ReceivedFrom interface{}
}
```

### `Blocks` {#Blocks}
See [`core/types/block.go#L391`](https://github.com/ethereum/go-ethereum/blob/master/core/types/block.go#L391).

```go
type Blocks []*Block
```

### `Fetcher`
See [`eth/fetcher/fetcher.go#L106-L146`](https://github.com/ethereum/go-ethereum/blob/master/eth/fetcher/fetcher.go#L106-L146).

```go
// Fetcher is responsible for accumulating block announcements from various peers
// and scheduling them for retrieval.
type Fetcher struct {
    // Various event channels
    notify chan *announce
    inject chan *inject

    blockFilter  chan chan []*types.Block
    headerFilter chan chan *headerFilterTask
    bodyFilter   chan chan *bodyFilterTask

    done chan common.Hash
    quit chan struct{}

    // Announce states
    announces  map[string]int              // Per peer announce counts to prevent memory exhaustion
    announced  map[common.Hash][]*announce // Announced blocks, scheduled for fetching
    fetching   map[common.Hash]*announce   // Announced blocks, currently fetching
    fetched    map[common.Hash][]*announce // Blocks with headers fetched, scheduled for body retrieval
    completing map[common.Hash]*announce   // Blocks with headers, currently body-completing

    // Block cache
    queue  *prque.Prque            // Queue containing the import operations (block number sorted)
    queues map[string]int          // Per peer block counts to prevent memory exhaustion
    queued map[common.Hash]*inject // Set of already queued blocks (to dedupe imports)

    // Callbacks
    getBlock       blockRetrievalFn   // Retrieves a block from the local chain
    verifyHeader   headerVerifierFn   // Checks if a block's headers have a valid proof of work
    broadcastBlock blockBroadcasterFn // Broadcasts a block to connected peers
    chainHeight    chainHeightFn      // Retrieves the current chain's height
    insertChain    chainInsertFn      // Injects a batch of blocks into the chain
    dropPeer       peerDropFn         // Drops a peer for misbehaving

    // Testing hooks
    announceChangeHook func(common.Hash, bool) // Method to call upon adding or deleting a hash from the announce list
    queueChangeHook    func(common.Hash, bool) // Method to call upon adding or deleting a block from the import queue
    fetchingHook       func([]common.Hash)     // Method to call upon starting a block (eth/61) or header (eth/62) fetch
    completingHook     func([]common.Hash)     // Method to call upon starting a block body fetch (eth/62)
    importedHook       func(*types.Block)      // Method to call upon successful block import (both eth/61 and eth/62)
}
```

### `prque.Prque`
A priority queue from [`karalabe/cookiejar.v2`](gopkg.in/karalabe/cookiejar.v2). See [`prque/prque.go#L26-L29`](https://github.com/karalabe/cookiejar/blob/master/collections/prque/prque.go#L26-L29)

```go
// Priority queue data structure.
type Prque struct {
    cont *sstack
}
```

### `Transaction`
See [`core/types/transaction.go#L38-L44`](https://github.com/ethereum/go-ethereum/blob/master/core/types/transaction.go#L38-L44).

```go
type Transaction struct {
    data txdata
    // caches
    hash atomic.Value
    size atomic.Value
    from atomic.Value
}
```

### `Transactions`
See [`core/types/transaction.go#L254-L255`](https://github.com/ethereum/go-ethereum/blob/master/core/types/transaction.go#L254-L255).

```go
// Transactions is a Transaction slice type for basic sorting.
type Transactions []*Transaction
```


### ``
See [``]().

```go
```

