# Types

The `go-ethereum` project defines 57 types. Only some of them are discussed in this documentation.

This section describes critical data structures used in `go-ethereum`. The main categories of types defined by this documentation are:

* [Common](common.md) &ndash; foundational types used throughout
* [Database](database.md) &ndash; defines blockchain data structures
* [Peer to Peer](p2p.md) &ndash; defines Ethereum nodes
* [Smart Contract](smart_contract.md) &ndash; defines data structures for smart contracts

## Public / Private Visibility
[Remember](https://tour.golang.org/basics/3), publicly visible methods and properties in Go (those that are _exported_) are denoted by having names that start with a capital letter. Private scope is denoted by names starting with a lower-case letter.

## Incantation: Counting Types
The following incantation reports the number of types in the `go-ethereum` project:

```bash
grep -rIhw --include \*.go "^\s*Type\s*" | \
  tr -d ',' | tr -d ':' | sed 's^//.*^^' | awk '{$1=$1};1' | \
  sort | uniq | wc -l
```

