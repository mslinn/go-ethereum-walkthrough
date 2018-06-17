## Packages {#packages}

`geth` defines 244 packages. Only the top-level packages are discussed here. The [`godoc`](https://godoc.org/github.com/ethereum/go-ethereum#pkg-subdirectories) for the project contains much of the following documentation for the top-level packages. The rest of the information was taken from disparate sources, including reading the source code:

Note: The [`build/`](https://github.com/ethereum/go-ethereum/tree/master/build) directory does not contain a Go source package; instead, it contains scripts and configurations for building the package in various environments.

| Directory | Description |
| --- | --- |
| [`accounts`](https://github.com/ethereum/go-ethereum/tree/master/accounts) | implements high-level Ethereum account management. |
| [`bmt`](https://github.com/ethereum/go-ethereum/tree/master/bmt) | provides a binary Merkle tree implementation. |
| [`cmd`](https://github.com/ethereum/go-ethereum/tree/master/cmd) | source for the command-line programs in the next table, below. |
| [`common`](https://github.com/ethereum/go-ethereum/tree/master/common) | contains various helper functions worth checking out |
| [`consensus`](https://github.com/ethereum/go-ethereum/tree/master/consensus) | implements different Ethereum consensus engines (which must conform to the [`Engine` interface](https://godoc.org/github.com/ethereum/go-ethereum/consensus#Engine)): [`clique`](https://godoc.org/github.com/ethereum/go-ethereum/consensus/clique) implements proof-of-authority consensus, and [`ethash`](https://godoc.org/github.com/ethereum/go-ethereum/consensus/ethash) implements proof-of-work consensus. |
| [`console`](https://github.com/ethereum/go-ethereum/tree/master/console) | Ethereum implements a JavaScript runtime environment (JSRE) that can be used in either interactive (console) or non-interactive (script) mode. Ethereum&#039;s JavaScript console exposes the full web3 JavaScript Dapp API and the admin API. [More documentation is here.](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) This package implements JSRE for the `geth attach` and `geth console` subcommands. |
| [`containers`](https://github.com/ethereum/go-ethereum/tree/master/containers) |  |
| [`contracts`](https://github.com/ethereum/go-ethereum/tree/master/contracts) |  |
| [`core`](https://github.com/ethereum/go-ethereum/tree/master/core) | implements the Ethereum consensus protocol, implements the Ethereum Virtual Machine, and other miscellaneous important bits |
| [`crypto`](https://github.com/ethereum/go-ethereum/tree/master/crypto) | cryptographic implementations |
| [`dashboard`](https://github.com/ethereum/go-ethereum/tree/master/dashboard) |  |
| [`eth`](https://github.com/ethereum/go-ethereum/tree/master/eth) | implements the Ethereum protocol |
| [`ethclient`](https://github.com/ethereum/go-ethereum/tree/master/ethclient) | provides a client for the Ethereum RPC API |
| [`ethdb`](https://github.com/ethereum/go-ethereum/tree/master/ethdb) |  |
| [`ethstats`](https://github.com/ethereum/go-ethereum/tree/master/ethstats) | implements the network stats reporting service |
| [`event`](https://github.com/ethereum/go-ethereum/tree/master/event) | deals with subscriptions to real-time events |
| [`internal`](https://github.com/ethereum/go-ethereum/tree/master/internal) | Debugging support, JavaScript dependencies, testing support |
| [`les`](https://github.com/ethereum/go-ethereum/tree/master/les) | implements the Light Ethereum Subprotocol |
| [`light`](https://github.com/ethereum/go-ethereum/tree/master/light) | implements on-demand retrieval capable state and chain objects for the Ethereum Light Client |
| [`log`](https://github.com/ethereum/go-ethereum/tree/master/log) | provides an opinionated, simple toolkit for best-practice logging that is both human and machine readable |
| [`metrics`](https://github.com/ethereum/go-ethereum/tree/master/metrics) | port of Coda Hale&#039;s Metrics library. Unclear why this was not implemented as a separate library, like [this one](https://github.com/rcrowley/go-metrics). |
| [`miner`](https://github.com/ethereum/go-ethereum/tree/master/miner) | implements Ethereum block creation and mining |
| [`mobile`](https://github.com/ethereum/go-ethereum/tree/master/mobile) | contains the simplified mobile APIs to go-ethereum |
| [`node`](https://github.com/ethereum/go-ethereum/tree/master/node) | sets up multi-protocol Ethereum nodes |
| [`p2p`](https://github.com/ethereum/go-ethereum/tree/master/p2p) | implements the Ethereum p2p network protocols: Node Discovery Protocol, RLPx v5 Topic Discovery Protocol, Ethereum Node Records as defined in EIP-778, common network port mapping protocols, and p2p network simulation. |
| [`params`](https://github.com/ethereum/go-ethereum/tree/master/params) |  |
| [`rlp`](https://github.com/ethereum/go-ethereum/tree/master/rlp) | implements the RLP serialization format |
| [`rpc`](https://github.com/ethereum/go-ethereum/tree/master/rpc) | provides access to the exported methods of an object across a network or other I/O connection |
| [`signer`](https://github.com/ethereum/go-ethereum/tree/master/signer) |  |
| [`swarm`](https://github.com/ethereum/go-ethereum/tree/master/swarm) |  |
| [`tests`](https://github.com/ethereum/go-ethereum/tree/master/tests) | implements execution of Ethereum JSON tests |
| [`trie`](https://github.com/ethereum/go-ethereum/tree/master/trie) | implements Merkle Patricia Tries |
| [`vendor`](https://github.com/ethereum/go-ethereum/tree/master/vendor) | contains a minimal framework for creating and organizing command line Go applications, and a rich testing extension for Go&#039;s testing package |
| [`whisper`](https://github.com/ethereum/go-ethereum/tree/master/whisper) | implements the Whisper protocol |

## Command-Line Programs
The [`cmd`](https://github.com/ethereum/go-ethereum/tree/master/cmd) directory contains source for the following command-line programs:

| Directory | Description |
| --- | --- |
| [`abigen`](https://github.com/ethereum/go-ethereum/tree/master/cmd/abigen) | source code generator to convert Ethereum contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [Ethereum contract ABIs](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. Please see the [Native DApps wiki page](https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts) for details. |
| [`bootnode`](https://github.com/ethereum/go-ethereum/tree/master/cmd/bootnode) | runs a bootstrap node for the Ethereum Discovery Protocol. This is a stripped-down version of geth that only takes part in the network node discovery protocol, and does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks.
| [`clef`](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef) | a standalone signer that manages keys across multiple Ethereum-aware apps such as Geth, Metamask, and cpp-ethereum. Alpha quality, not released yet. 
| [`ethkey`](https://github.com/ethereum/go-ethereum/tree/master/cmd/ethkey) | a key/wallet management tool for Ethereum keys. Allows user to add, remove and change their keys, and supports cold wallet device-friendly transaction inspection and signing. [This documentation](https://github.com/ethereum/guide/blob/master/ethkey.md) was written for the C++ Ethereum client implementation, but it is probably suitable for the Go implementation as well. 
| [`evm`](https://github.com/ethereum/go-ethereum/tree/master/cmd/evm) | a version of the EVM (Ethereum Virtual Machine) for running bytecode snippets within a configurable environment and execution mode. Allows isolated, fine-grained debugging of EVM opcodes. <pre>evm --code 60ff60ff --debug</pre>
| [`faucet`](https://github.com/ethereum/go-ethereum/tree/master/cmd/faucet) | an Ether faucet backed by a light client. 
| [`geth`](https://github.com/ethereum/go-ethereum/tree/master/cmd/geth) | official command-line client for Ethereum. It provides the entry point into the Ethereum network (main-, test- or private net), capable of running as a full node (default) archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Ethereum network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. For more information see the [CLI Wiki page.](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)
| [`p2psim`](https://github.com/ethereum/go-ethereum/tree/master/cmd/p2psim) | a simulation HTTP API.
| [`puppeth`](https://github.com/ethereum/go-ethereum/tree/master/cmd/puppeth) | assembles and maintains private networks. 
| [`rlpdump`](https://github.com/ethereum/go-ethereum/tree/master/cmd/rlpdump) | a pretty-printer for RLP data. RLP (Recursive Length Prefix) is the data encoding used by the Ethereum protocol. Sample usage: <pre>rlpdump --hex CE0183FFFFFFC4C304050583616263</pre>
| [`swarm`](https://github.com/ethereum/go-ethereum/tree/master/cmd/swarm) | provides the `bzzhash` command, which computes a swarm tree hash, and implements the swarm daemon and tools. See the [swarm documentation](https://swarm-guide.readthedocs.io/) for more information. 
| [`wnode`](https://github.com/ethereum/go-ethereum/tree/master/cmd/wnode) | simple Whisper node. It could be used as a stand-alone bootstrap node. Also could be used for different test and diagnostics purposes. |

## Incantations
The following incantation shows that the Ethereum project defines 244 packages:

```
grep -rh "^package" | grep -v "not installed" | \
  tr -d ';' | sed 's^//.*^^' | awk '{$1=$1};1' | \
  sort | uniq | wc -l
```

The following incantation lists the top-level package names:

```find . -maxdepth 1 -type d | sed 's^\./^^' | sed '/\..*/d'```

