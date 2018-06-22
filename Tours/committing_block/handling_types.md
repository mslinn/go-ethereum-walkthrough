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

### `peer` {#peer}
This private type has only private members; see [`eth/peer.go#L75-L94`](https://github.com/ethereum/go-ethereum/blob/master/eth/peer.go#L75-L94).

```go
type peer struct {
id string

*p2p.Peer
rw p2p.MsgReadWriter

version int // Protocol version negotiated
forkDrop *time.Timer // Timed connection dropper if forks aren't validated in time

head common.Hash
td *big.Int
lock sync.RWMutex

knownTxs *set.Set // Set of transaction hashes known to be known by this peer
knownBlocks *set.Set // Set of block hashes known to be known by this peer
queuedTxs chan []*types.Transaction // Queue of transactions to broadcast to the peer
queuedProps chan *propEvent // Queue of blocks to broadcast to the peer
queuedAnns chan *types.Block // Queue of blocks to announce to the peer
term chan struct{} // Termination channel to stop the broadcaster
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

### `BlockChain` {#BlockChain}
See [`core/blockchain.go#L75-L132`](https://github.com/ethereum/go-ethereum/blob/master/core/blockchain.go#L75-L132).

```go
// BlockChain represents the canonical chain given a database with a genesis
// block. The Blockchain manages chain imports, reverts, chain reorganisations.
//
// Importing blocks in to the block chain happens according to the set of rules
// defined by the two stage Validator. Processing of blocks is done using the
// Processor which processes the included transaction. The validation of the state
// is done in the second part of the Validator. Failing results in aborting of
// the import.
//
// The BlockChain also helps in returning blocks from **any** chain included
// in the database as well as blocks that represents the canonical chain. It's
// important to note that GetBlock can return any block and does not need to be
// included in the canonical one where as GetBlockByNumber always represents the
// canonical chain.
type BlockChain struct {
    chainConfig *params.ChainConfig // Chain & network configuration
    cacheConfig *CacheConfig        // Cache configuration for pruning

    db     ethdb.Database // Low level persistent database to store final content in
    triegc *prque.Prque   // Priority queue mapping block numbers to tries to gc
    gcproc time.Duration  // Accumulates canonical block processing for trie dumping

    hc            *HeaderChain
    rmLogsFeed    event.Feed
    chainFeed     event.Feed
    chainSideFeed event.Feed
    chainHeadFeed event.Feed
    logsFeed      event.Feed
    scope         event.SubscriptionScope
    genesisBlock  *types.Block

    mu      sync.RWMutex // global mutex for locking chain operations
    chainmu sync.RWMutex // blockchain insertion lock
    procmu  sync.RWMutex // block processor lock

    checkpoint       int          // checkpoint counts towards the new checkpoint
    currentBlock     atomic.Value // Current head of the block chain
    currentFastBlock atomic.Value // Current head of the fast-sync chain (may be above the block chain!)

    stateCache   state.Database // State database to reuse between imports (contains state cache)
    bodyCache    *lru.Cache     // Cache for the most recent block bodies
    bodyRLPCache *lru.Cache     // Cache for the most recent block bodies in RLP encoded format
    blockCache   *lru.Cache     // Cache for the most recent entire blocks
    futureBlocks *lru.Cache     // future blocks are blocks added for later processing

    quit    chan struct{} // blockchain quit channel
    running int32         // running must be called atomically
    // procInterrupt must be atomically called
    procInterrupt int32          // interrupt signaler for block processing
    wg            sync.WaitGroup // chain processing wait group for shutting down

    engine    consensus.Engine
    processor Processor // block processor interface
    validator Validator // block and state validator interface
    vmConfig  vm.Config

    badBlocks *lru.Cache // Bad block cache
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
