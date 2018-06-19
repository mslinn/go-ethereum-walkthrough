# Blockchain Database {#blockchain}
These types define the blockchain database.

## Trie {#trie}
[Wikipedia says](https://en.wikipedia.org/wiki/Trie): 

> a trie, also called digital tree and sometimes radix tree or prefix tree (as they can be searched by prefixes), is a kind of search tree—an ordered tree data structure that is used to store a dynamic set or associative array where the keys are usually strings. Unlike a binary search tree, no node in the tree stores the key associated with that node; instead, its position in the tree defines the key with which it is associated. All the descendants of a node have a common prefix of the string associated with that node, and the root is associated with the empty string. Values are not necessarily associated with every node. Rather, values tend only to be associated with leaves, and with some inner nodes that correspond to keys of interest. For the space-optimized presentation of prefix tree, see compact prefix tree.

_More to come..._

## Database {#db}
`go-ethereum` provides the `Database` type, which encapsulates `Trie`; see [`core/state/database.go#L41-L60`](https://github.com/ethereum/go-ethereum/blob/master/core/state/database.go#L41-L60).
```go
// Database wraps access to tries and contract code.
type Database interface {
	// OpenTrie opens the main account trie.
	OpenTrie(root common.Hash) (Trie, error)

	// OpenStorageTrie opens the storage trie of an account.
	OpenStorageTrie(addrHash, root common.Hash) (Trie, error)

	// CopyTrie returns an independent copy of the given trie.
	CopyTrie(Trie) Trie

	// ContractCode retrieves a particular contract's code.
	ContractCode(addrHash, codeHash common.Hash) ([]byte, error)

	// ContractCodeSize retrieves a particular contracts code's size.
	ContractCodeSize(addrHash, codeHash common.Hash) (int, error)

	// TrieDB retrieves the low level trie database used for data storage.
	TrieDB() *trie.Database
}
```

_More to come..._

## Merkle Trie {#merkle}
The `go-ethereum` project calls this data structure a `Trie`, whereas Wikipedia calls it a `tree`. This document follows the `go-ethereum` convention.

[Wikipedia says](https://en.wikipedia.org/wiki/Merkle_tree):
> In cryptography and computer science, a hash tree or Merkle tree is a tree in which every leaf node is labelled with the hash of a data block and every non-leaf node is labelled with the cryptographic hash of the labels of its child nodes. Hash trees allow efficient and secure verification of the contents of large data structures. Hash trees are a generalization of hash lists and hash chains.

> Demonstrating that a leaf node is a part of a given binary hash tree requires computing a number of hashes proportional to the logarithm of the number of leaf nodes of the tree; this contrasts with hash lists, where the number is proportional to the number of leaf nodes itself.

`go-ethereum` defines the following `Trie interface`; see [`core/state/database.go#L62-L72`](https://github.com/ethereum/go-ethereum/blob/master/core/state/database.go#L62-L72)
```go
// Trie is a Ethereum Merkle Trie.
type Trie interface {
	TryGet(key []byte) ([]byte, error)
	TryUpdate(key, value []byte) error
	TryDelete(key []byte) error
	Commit(onleaf trie.LeafCallback) (common.Hash, error)
	Hash() common.Hash
	NodeIterator(startKey []byte) trie.NodeIterator
	GetKey([]byte) []byte // TODO(fjl): remove this when SecureTrie is removed
	Prove(key []byte, fromLevel uint, proofDb ethdb.Putter) error
}
```

The same file defines the [`cachingDB`](https://github.com/ethereum/go-ethereum/blob/master/core/state/database.go#L62-L72) type, which is concurrency aware:
```go
type cachingDB struct {
	db            *trie.Database
	mu            sync.Mutex
	pastTries     []*trie.SecureTrie
	codeSizeCache *lru.Cache
}
```

Package [`bmt`](https://godoc.org/github.com/ethereum/go-ethereum/bmt) in the `go-ethereum` project provides a binary Merkle tree implementation.

[`core/state/statedb.go#L47-L84`](https://github.com/ethereum/go-ethereum/blob/master/core/state/statedb.go#L47-L84) says:
```go
// StateDBs within the ethereum protocol are used to store anything
// within the merkle trie. StateDBs take care of caching and storing
// nested states. It's the general query interface to retrieve:
// * Contracts
// * Accounts
type StateDB struct {
	db   Database
	trie Trie

	// This map holds 'live' objects, which will get modified while processing a state transition.
	stateObjects      map[common.Address]*stateObject
	stateObjectsDirty map[common.Address]struct{}

	// DB error.
	// State objects are used by the consensus core and VM which are
	// unable to deal with database-level errors. Any error that occurs
	// during a database read is memoized here and will eventually be returned
	// by StateDB.Commit.
	dbErr error

	// The refund counter, also used by state transitioning.
	refund uint64

	thash, bhash common.Hash
	txIndex      int
	logs         map[common.Hash][]*types.Log
	logSize      uint

	preimages map[common.Hash][]byte

	// Journal of state modifications. This is the backbone of
	// Snapshot and RevertToSnapshot.
	journal        *journal
	validRevisions []revision
	nextRevisionId int

	lock sync.Mutex
}
```
Methods to manipulate `StateDB` follow the `struct`, such as `New`, `Error`, `Reset`, etc.

_More to come..._

## Patricia Trie {#patricia}
Package [`trie`](https://godoc.org/github.com/ethereum/go-ethereum/trie) implements a Patricia trie.

[Wikipedia says](https://en.wikipedia.org/wiki/Radix_tree): 
> A radix tree (also radix trie or compact prefix tree) is a data structure that represents a space-optimized trie in which each node that is the only child is merged with its parent. The result is that the number of children of every internal node is at most the radix r of the radix tree, where r is a positive integer and a power x of 2, having x ≥ 1. Unlike in regular tries, edges can be labeled with sequences of elements as well as single elements. This makes radix trees much more efficient for small sets (especially if the strings are long) and for sets of strings that share long prefixes.

> Unlike regular trees (where whole keys are compared en masse from their beginning up to the point of inequality), the key at each node is compared chunk-of-bits by chunk-of-bits, where the quantity of bits in that chunk at that node is the radix r of the radix trie. When the r is 2, the radix trie is binary (i.e., compare that node's 1-bit portion of the key), which minimizes sparseness at the expense of maximizing trie depth—i.e., maximizing up to conflation of nondiverging bit-strings in the key. When r is an integer power of 2 greater or equal to 4, then the radix trie is an r-ary trie, which lessens the depth of the radix trie at the expense of potential sparseness.

> As an optimization, edge labels can be stored in constant size by using two pointers to a string (for the first and last elements).[1]

> Donald R. Morrison first described what he called "Patricia trees" in 1968; the name comes from the acronym PATRICIA, which stands for "Practical Algorithm To Retrieve Information Coded In Alphanumeric". Gernot Gwehenberger independently invented and described the data structure at about the same time. PATRICIA trees are radix trees with radix equals 2, which means that each bit of the key is compared individually and each node is a two-way (i.e., left versus right) branch.

_More to come..._