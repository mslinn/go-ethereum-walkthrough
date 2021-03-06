## Packages {#packages}

The [`go-ethereum`](https://github.com/ethereum/go-ethereum) project defines 242 packages. Only the top-level packages are discussed here. The [`godoc`](https://godoc.org/github.com/ethereum/go-ethereum#pkg-subdirectories) for the project contains some of the following documentation for the top-level packages. The rest of the information was taken from disparate sources, including the [wiki](https://github.com/ethereum/go-ethereum/wiki) and reading the source code.

### Notes {#notes}

1. The [`build/`](https://github.com/ethereum/go-ethereum/tree/master/build) directory only contains one special Go command-line program package, for building the distribution. The `build` directory also contains scripts and configurations for building the package in various environments.

2. Only the directories that map 1:1 with packages are shown in the following table. The exception is the `ethereum` package, which is defined in only one file: [`interfaces.go`](https://github.com/ethereum/go-ethereum/blob/master/interfaces.go). There is a note in that file to migrate the contents to the `event` package.

3. Go&rsquo;s [special treatment](https://blog.gopheracademy.com/advent-2015/vendor-folder/) of the [`vendor/`](https://github.com/ethereum/go-ethereum/tree/master/vendor) directory defines dependencies which contain a minimal framework for creating and organizing command line Go applications, and a rich testing extension for Go&rsquo;s testing package. [Details are here](https://preview.tinyurl.com/y7hnr6w3).

### Top-Level Packages {#tlp}

| Directory | Description |
| --- | --- |
| [`accounts`](https://github.com/ethereum/go-ethereum/tree/master/accounts) | Implements high-level Ethereum account management. |
| [`bmt`](https://github.com/ethereum/go-ethereum/tree/master/bmt) | Provides a binary Merkle tree implementation. |
| [`cmd`](https://github.com/ethereum/go-ethereum/tree/master/cmd) | Source for the command-line programs in the next table, below. |
| [`common`](https://github.com/ethereum/go-ethereum/tree/master/common) | Defines helper functions: [`bitutil`](https://godoc.org/github.com/ethereum/go-ethereum/common/bitutil), which implements fast bitwise operations; [`compiler`](https://godoc.org/github.com/ethereum/go-ethereum/common/compiler), which wraps the Solidity compiler executable (`solc`); [`hexutil`](https://godoc.org/github.com/ethereum/go-ethereum/common/hexutil), which implements hex encoding with `0x` prefix; [`math`](https://godoc.org/github.com/ethereum/go-ethereum/common/math), which provides integer math functions; and [`mclock`](https://godoc.org/github.com/ethereum/go-ethereum/common/mclock), a wrapper for a monotonic clock source. |
| [`consensus`](https://github.com/ethereum/go-ethereum/tree/master/consensus) | Implements different Ethereum consensus engines (which must conform to the [`Engine` interface](https://godoc.org/github.com/ethereum/go-ethereum/consensus#Engine): [`clique`](https://godoc.org/github.com/ethereum/go-ethereum/consensus/clique), which implements proof-of-authority consensus, and [`ethash`](https://godoc.org/github.com/ethereum/go-ethereum/consensus/ethash), which implements proof-of-work consensus. The original Ethash docs are [here.](https://github.com/ethereum/wiki/wiki/Ethash) |
| [`console`](https://github.com/ethereum/go-ethereum/tree/master/console) | Ethereum implements a JavaScript runtime environment (JSRE) that can be used in either interactive (console) or non-interactive (script) mode. Ethereum&#039;s JavaScript console exposes the full web3 JavaScript Dapp API and the admin API. [More documentation is here.](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console) This package implements JSRE for the `geth attach` and `geth console` subcommands. |
| [`containers`](https://github.com/ethereum/go-ethereum/tree/master/containers) | Currently just provides docker support. |
| [`contracts`](https://github.com/ethereum/go-ethereum/tree/master/contracts) | Supports the [`chequebook` smart contract](https://godoc.org/github.com/ethereum/go-ethereum/contracts/chequebook) and the Ethereum Name Service ([EIP 137](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md)). |
| [`core`](https://github.com/ethereum/go-ethereum/tree/master/core) | Implements the Ethereum consensus protocol, implements the Ethereum Virtual Machine, and other miscellaneous important bits. |
| [`crypto`](https://github.com/ethereum/go-ethereum/tree/master/crypto) | Cryptographic implementations. |
| [`dashboard`](https://github.com/ethereum/go-ethereum/tree/master/dashboard) | Client-server data visualizer integrated into geth, that collects and visualizes information about an Ethereum node. |
| [`eth`](https://github.com/ethereum/go-ethereum/tree/master/eth) | Implements the Ethereum protocol. |
| [`ethclient`](https://github.com/ethereum/go-ethereum/tree/master/ethclient) | Provides a client for the Ethereum RPC API. |
| [`ethdb`](https://github.com/ethereum/go-ethereum/tree/master/ethdb) | `TODO` leveldb code? |
| [`ethstats`](https://github.com/ethereum/go-ethereum/tree/master/ethstats) | Implements the network stats reporting service. |
| [`event`](https://github.com/ethereum/go-ethereum/tree/master/event) | Deals with subscriptions to real-time events. |
| [`internal`](https://github.com/ethereum/go-ethereum/tree/master/internal) | Debugging support, JavaScript dependencies, testing support. |
| [`les`](https://github.com/ethereum/go-ethereum/tree/master/les) | Implements the Light Ethereum Subprotocol. |
| [`light`](https://github.com/ethereum/go-ethereum/tree/master/light) | Implements on-demand retrieval capable state and chain objects for the [Ethereum Light Client](https://github.com/ethereum/wiki/wiki/Light-client-protocol). |
| [`log`](https://github.com/ethereum/go-ethereum/tree/master/log) | Provides an opinionated, simple toolkit for best-practice logging that is both human and machine readable. |
| [`metrics`](https://github.com/ethereum/go-ethereum/tree/master/metrics) | Port of Coda Hale&#039;s Metrics library. `Suggestion`: Why was this not implemented as a separate library, like [this one](https://github.com/rcrowley/go-metrics)? |
| [`miner`](https://github.com/ethereum/go-ethereum/tree/master/miner) | Implements Ethereum block creation and mining. |
| [`mobile`](https://github.com/ethereum/go-ethereum/tree/master/mobile) | Contains the simplified mobile APIs to go-ethereum. |
| [`node`](https://github.com/ethereum/go-ethereum/tree/master/node) | Sets up multi-protocol Ethereum nodes. |
| [`p2p`](https://github.com/ethereum/go-ethereum/tree/master/p2p) | Implements the Ethereum p2p network protocols: Node Discovery Protocol, RLPx v5 Topic Discovery Protocol, Ethereum Node Records as defined in [EIP-778](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-778.md), common network port mapping protocols, and p2p network simulation. |
| [`params`](https://github.com/ethereum/go-ethereum/tree/master/params) | TODO what is this? |
| [`rlp`](https://github.com/ethereum/go-ethereum/tree/master/rlp) | Implements the RLP serialization format. |
| [`rpc`](https://github.com/ethereum/go-ethereum/tree/master/rpc) | Provides access to the exported methods of an object across a network or other I/O connection. |
| [`signer`](https://github.com/ethereum/go-ethereum/tree/master/signer) | Rule-based transaction signer. |
| [`swarm`](https://github.com/ethereum/go-ethereum/tree/master/swarm) | [Swarm is a service](https://blog.ethereum.org/2018/06/21/announcing-swarm-proof-of-concept-release-3/) that provides APIs to upload and download content to the cloud and through URL-based addressing offers virtual hosting of websites and decentralised applications (dapps) without web servers, using decentralised peer-to-peer distributed infrastructure. |
| [`tests`](https://github.com/ethereum/go-ethereum/tree/master/tests) | Implements execution of Ethereum JSON tests. |
| [`trie`](https://github.com/ethereum/go-ethereum/tree/master/trie) | Implements [Merkle Patricia tries](https://github.com/ethereum/wiki/wiki/%5BEnglish%5D-Patricia-Tree). |
| [`whisper`](https://github.com/ethereum/go-ethereum/tree/master/whisper) | Implements the [Whisper protocol](https://github.com/ethereum/wiki/wiki/Whisper). |

### Command-Line Programs {#cli}
The [`cmd`](https://github.com/ethereum/go-ethereum/tree/master/cmd) directory contains source for the following command-line programs:

| Directory | Description |
| --- | --- |
| [`abigen`](https://github.com/ethereum/go-ethereum/tree/master/cmd/abigen) | Source code generator to convert Ethereum contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [Ethereum contract ABIs](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. See the [Native DApps wiki page](https://github.com/ethereum/go-ethereum/wiki/Native-DApps:-Go-bindings-to-Ethereum-contracts) for details. |
| [`bootnode`](https://github.com/ethereum/go-ethereum/tree/master/cmd/bootnode) | Runs a bootstrap node for the Ethereum Discovery Protocol. This is a stripped-down version of `geth` that only takes part in the network node discovery protocol, and does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks.
| [`clef`](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef) | A standalone signer that manages keys across multiple Ethereum-aware apps such as `geth`, Metamask, and `cpp-ethereum`. This command-line program is currently only alpha quality, and has not been formally released yet. 
| [`ethkey`](https://github.com/ethereum/go-ethereum/tree/master/cmd/ethkey) | A key/wallet management tool for Ethereum keys. Allows user to add, remove and change their keys, and supports cold wallet device-friendly transaction inspection and signing. [This documentation](https://github.com/ethereum/guide/blob/master/ethkey.md) was written for the C++ Ethereum client implementation, but it is probably suitable for the Go implementation as well. 
| [`evm`](https://github.com/ethereum/go-ethereum/tree/master/cmd/evm) | A version of the EVM (Ethereum Virtual Machine) for running bytecode snippets within a configurable environment and execution mode. Allows isolated, fine-grained debugging of EVM opcodes. <pre>evm --code 60ff60ff --debug</pre>
| [`faucet`](https://github.com/ethereum/go-ethereum/tree/master/cmd/faucet) | An Ether faucet backed by a light client. `TODO` is this actually a subcommand of another command? 
| [`geth`](https://github.com/ethereum/go-ethereum/tree/master/cmd/geth) | Official command-line client for Ethereum. It provides the entry point into the Ethereum network (main-, test- or private net), capable of running as a full node (default) archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Ethereum network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. Command-line options are described on the [CLI Wiki page.](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)
| [`p2psim`](https://github.com/ethereum/go-ethereum/tree/master/cmd/p2psim) | A simulation HTTP API.
| [`puppeth`](https://github.com/ethereum/go-ethereum/tree/master/cmd/puppeth) | Assembles and maintains new private Ethereum networks, including genesis, bootnodes, signers, ethstats, faucet, and dashboard. [Puppeth](https://blog.ethereum.org/2017/04/14/geth-1-6-puppeth-master/) uses `ssh` to dial into remote servers, and builds its network components out of docker containers using `docker-compose`. The gitter room for puppeth is [`ethereum/puppeth`](https://gitter.im/ethereum/puppeth).
| [`rlpdump`](https://github.com/ethereum/go-ethereum/tree/master/cmd/rlpdump) | A pretty-printer for RLP data. RLP (Recursive Length Prefix) is the data encoding used by the Ethereum protocol. Sample usage: <pre>rlpdump --hex CE0183FFFFFFC4C304050583616263</pre>
| [`swarm`](https://github.com/ethereum/go-ethereum/tree/master/cmd/swarm) | Provides the `bzzhash` command, which computes a swarm tree hash, and implements the swarm daemon and tools. See the [swarm documentation](https://swarm-guide.readthedocs.io/) for more information. 
| [`wnode`](https://github.com/ethereum/go-ethereum/tree/master/cmd/wnode) | Whisper node, which could be used as a stand-alone bootstrap node and could also be used for test and diagnostics purposes. |

### Incantations {#incantations}

#### Counting Packages {#counting}
The following incantation reports the number of packages in the `go-ethereum` project:

```bash
grep -rIhw --include \*.go "^\s*package\s*" | grep -v "not installed" | \
  tr -d ';' | sed 's^//.*^^' | awk '{$1=$1};1' | \
  sort | uniq | wc -l
```

#### Counting Top-Level Directories {#tld}
The following incantation lists the top-level directories, most of which are package names:

```bash
find . -maxdepth 1 -type d | sed 's^\./^^' | sed '/\..*/d'
```

