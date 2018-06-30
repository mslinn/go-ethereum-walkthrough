# Tours

## Foreward {#forward}

You can just passively read this document and gain lots of insight into how Ethereum works. The next level of understanding happens when you trace through the source code yourself according to what is written here. If and when you are ready to do that, read this section. Otherwise, just know that the information for how the walkthroughs were gathered is documented in this section and move lightly along to any of the walkthroughs that interest you of **GoEthereum Walkthrough**.

## Setup {#setup}

These instructions set up a new private, live Ethereum network with pre-allocated funds in the default account. Because this is a new network, with no previous transactions to load, it boots up immediately. The balance in the default account for this network will be initialized when the genesis block is created. This is the easiest way to set up an account with a non-zero balance.

Because we want to set run a debugger on `geth` and breakpoints, we must first download the `go-ethereum` source code, fetch all the dependencies, and build the `geth` command.

### Check out and Build `go-ethereum` {#checkout}
1. Ensure that [`GOPATH`](https://github.com/golang/go/wiki/GOPATH) is set:
  ```bash
  $ echo $GOPATH
  /home/mslinn/go
  ```
  
2. Create the directory to hold `go-ethereum`. It must be exactly this name:
  ```bash
  $ mkdir -p $GOPATH/src/github.com/ethereum
  $ cd $GOPATH/src/github.com/ethereum/
  ```

3. Check out `go-ethereum`:
  ```bash
  $ git clone git@github.com:ethereum/go-ethereum.git go-ethereum
  $ cd go-ethereum
  ```
  
4. The `go-ethereum` project has a `Makefile` so `make` can be used to build all the command line programs in `go-ethereum`, or selected command line programs. The `Makefile` calls `build/ci.go`, which does not have any documentation or help except that which is shown in the `godoc`:
  ```
  The ci command is called from Continuous Integration scripts.
  Usage: go run build/ci.go <command> <command flags/arguments>
  Available commands are:
     install    [ -arch architecture ] [ -cc compiler ] [ packages... ]                          -- builds packages and executables
     test       [ -coverage ] [ packages... ]                                                    -- runs the tests
     lint                                                                                        -- runs certain pre-selected linters
     archive    [ -arch architecture ] [ -type zip|tar ] [ -signer key-envvar ] [ -upload dest ] -- archives build artefacts
     importkeys                                                                                  -- imports signing keys from env
     debsrc     [ -signer key-id ] [ -upload dest ]                                              -- creates a debian source package
     nsis                                                                                        -- creates a Windows NSIS installer
     aar        [ -local ] [ -sign key-id ] [-deploy repo] [ -upload dest ]                      -- creates an Android archive
     xcode      [ -local ] [ -sign key-id ] [-deploy repo] [ -upload dest ]                      -- creates an iOS XCode framework
     xgo        [ -alltools ] [ options ]                                                        -- cross builds according to options
     purge      [ -store blobstore ] [ -days threshold ]                                         -- purges old archives from the blobstore
  For all commands, -n prevents execution of external programs (dry run mode).
  ```

  You can invoke the command that `make geth` actually runs like this:
  ```bash
  $ build/env.sh go run build/ci.go install ./cmd/geth
  >>> /usr/lib/go-1.10/bin/go install -ldflags -X main.gitCommit=e916f9786dd318ec873cab21c8092d4da2c8dd54 -v ./cmd/geth
  github.com/ethereum/go-ethereum/vendor/github.com/hashicorp/golang-lru/simplelru
  github.com/ethereum/go-ethereum/vendor/golang.org/x/net/html/atom
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/internal/identifier
  github.com/ethereum/go-ethereum/crypto/sha3
  github.com/ethereum/go-ethereum/vendor/golang.org/x/sys/unix
  github.com/ethereum/go-ethereum/common/hexutil
  github.com/ethereum/go-ethereum/vendor/github.com/go-stack/stack
  github.com/ethereum/go-ethereum/common/math
  github.com/ethereum/go-ethereum/rlp
  github.com/ethereum/go-ethereum/common
  github.com/ethereum/go-ethereum/crypto/secp256k1
  github.com/ethereum/go-ethereum/params
  github.com/ethereum/go-ethereum/log
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/util
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/cache
  github.com/ethereum/go-ethereum/vendor/github.com/elastic/gosigar
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/comparer
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/storage
  github.com/ethereum/go-ethereum/metrics
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/filter
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/opt
  github.com/ethereum/go-ethereum/vendor/github.com/golang/snappy
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/errors
  github.com/ethereum/go-ethereum/vendor/gopkg.in/karalabe/cookiejar.v2/collections/prque
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/iterator
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/journal
  github.com/ethereum/go-ethereum/vendor/github.com/aristanetworks/goarista/monotime
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/memdb
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb/table
  github.com/ethereum/go-ethereum/common/mclock
  github.com/ethereum/go-ethereum/event
  github.com/ethereum/go-ethereum/crypto/randentropy
  github.com/ethereum/go-ethereum/vendor/github.com/syndtr/goleveldb/leveldb
  github.com/ethereum/go-ethereum/vendor/github.com/rjeczalik/notify
  github.com/ethereum/go-ethereum/vendor/github.com/pborman/uuid
  github.com/ethereum/go-ethereum/vendor/golang.org/x/crypto/pbkdf2
  github.com/ethereum/go-ethereum/vendor/golang.org/x/crypto/scrypt
  github.com/ethereum/go-ethereum/vendor/gopkg.in/fatih/set.v0
  github.com/ethereum/go-ethereum/cmd/internal/browser
  github.com/ethereum/go-ethereum/common/fdlimit
  github.com/ethereum/go-ethereum/vendor/github.com/hashicorp/golang-lru
  github.com/ethereum/go-ethereum/p2p/netutil
  github.com/ethereum/go-ethereum/vendor/golang.org/x/net/context
  github.com/ethereum/go-ethereum/vendor/github.com/edsrzf/mmap-go
  github.com/ethereum/go-ethereum/vendor/github.com/rs/xhandler
  github.com/ethereum/go-ethereum/vendor/golang.org/x/net/websocket
  github.com/ethereum/go-ethereum/vendor/github.com/rs/cors
  github.com/ethereum/go-ethereum/common/bitutil
  github.com/ethereum/go-ethereum/ethdb
  github.com/ethereum/go-ethereum/crypto/bn256/cloudflare
  github.com/ethereum/go-ethereum/rpc
  github.com/ethereum/go-ethereum/vendor/golang.org/x/crypto/ripemd160
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/httpu
  github.com/ethereum/go-ethereum/crypto/bn256
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/scpd
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/soap
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/ssdp
  github.com/ethereum/go-ethereum/vendor/golang.org/x/net/html
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/transform
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/internal/utf8internal
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/runes
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/internal
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/charmap
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/japanese
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/korean
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/simplifiedchinese
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/traditionalchinese
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/unicode
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/internal/tag
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/language
  github.com/ethereum/go-ethereum/vendor/github.com/jackpal/go-nat-pmp
  github.com/ethereum/go-ethereum/vendor/github.com/davecgh/go-spew/spew
  github.com/ethereum/go-ethereum/eth/tracers/internal/tracers
  github.com/ethereum/go-ethereum/vendor/github.com/golang/protobuf/proto
  github.com/ethereum/go-ethereum/vendor/gopkg.in/olebedev/go-duktape.v3
  github.com/ethereum/go-ethereum/vendor/golang.org/x/text/encoding/htmlindex
  github.com/ethereum/go-ethereum/vendor/golang.org/x/net/html/charset
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/dcps/internetgateway1
  github.com/ethereum/go-ethereum/vendor/github.com/huin/goupnp/dcps/internetgateway2
  github.com/ethereum/go-ethereum/vendor/github.com/golang/protobuf/protoc-gen-go/descriptor
  github.com/ethereum/go-ethereum/accounts/usbwallet/internal/trezor
  github.com/ethereum/go-ethereum/p2p/nat
  github.com/ethereum/go-ethereum/vendor/github.com/karalabe/hid
  github.com/ethereum/go-ethereum/log/term
  github.com/ethereum/go-ethereum/metrics/exp
  github.com/ethereum/go-ethereum/vendor/github.com/fjl/memsize
  github.com/ethereum/go-ethereum/vendor/github.com/fjl/memsize/memsizeui
  github.com/ethereum/go-ethereum/vendor/github.com/mattn/go-colorable
  github.com/ethereum/go-ethereum/vendor/gopkg.in/urfave/cli.v1
  github.com/ethereum/go-ethereum/internal/debug
  github.com/ethereum/go-ethereum/vendor/github.com/prometheus/prometheus/util/flock
  github.com/ethereum/go-ethereum/les/flowcontrol
  github.com/ethereum/go-ethereum/vendor/golang.org/x/sync/syncmap
  github.com/ethereum/go-ethereum/internal/jsre/deps
  github.com/ethereum/go-ethereum/vendor/github.com/mattn/go-isatty
  github.com/ethereum/go-ethereum/vendor/github.com/fatih/color
  github.com/ethereum/go-ethereum/vendor/gopkg.in/sourcemap.v1/base64vlq
  github.com/ethereum/go-ethereum/vendor/gopkg.in/sourcemap.v1
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/file
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/token
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/ast
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/dbg
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/parser
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto/registry
  github.com/ethereum/go-ethereum/vendor/github.com/robertkrimen/otto
  github.com/ethereum/go-ethereum/internal/jsre
  github.com/ethereum/go-ethereum/internal/web3ext
  github.com/ethereum/go-ethereum/vendor/github.com/peterh/liner
  github.com/ethereum/go-ethereum/vendor/github.com/maruel/panicparse/stack
  github.com/ethereum/go-ethereum/vendor/github.com/mattn/go-runewidth
  github.com/ethereum/go-ethereum/vendor/github.com/mitchellh/go-wordwrap
  github.com/ethereum/go-ethereum/vendor/github.com/nsf/termbox-go
  github.com/ethereum/go-ethereum/vendor/github.com/gizak/termui
  github.com/ethereum/go-ethereum/vendor/github.com/naoina/go-stringutil
  github.com/ethereum/go-ethereum/vendor/github.com/naoina/toml/ast
  github.com/ethereum/go-ethereum/vendor/github.com/naoina/toml
  github.com/ethereum/go-ethereum/crypto
  github.com/ethereum/go-ethereum/trie
  github.com/ethereum/go-ethereum/crypto/ecies
  github.com/ethereum/go-ethereum/p2p/discover
  github.com/ethereum/go-ethereum/p2p/discv5
  github.com/ethereum/go-ethereum/core/types
  github.com/ethereum/go-ethereum
  github.com/ethereum/go-ethereum/core/state
  github.com/ethereum/go-ethereum/accounts
  github.com/ethereum/go-ethereum/core/rawdb
  github.com/ethereum/go-ethereum/accounts/keystore
  github.com/ethereum/go-ethereum/core/vm
  github.com/ethereum/go-ethereum/consensus
  github.com/ethereum/go-ethereum/consensus/misc
  github.com/ethereum/go-ethereum/consensus/clique
  github.com/ethereum/go-ethereum/consensus/ethash
  github.com/ethereum/go-ethereum/p2p
  github.com/ethereum/go-ethereum/core/bloombits
  github.com/ethereum/go-ethereum/core
  github.com/ethereum/go-ethereum/eth/fetcher
  github.com/ethereum/go-ethereum/accounts/usbwallet
  github.com/ethereum/go-ethereum/dashboard
  github.com/ethereum/go-ethereum/node
  github.com/ethereum/go-ethereum/eth/downloader
  github.com/ethereum/go-ethereum/eth/filters
  github.com/ethereum/go-ethereum/light
  github.com/ethereum/go-ethereum/whisper/whisperv6
  github.com/ethereum/go-ethereum/console
  github.com/ethereum/go-ethereum/internal/ethapi
  github.com/ethereum/go-ethereum/miner
  github.com/ethereum/go-ethereum/ethclient
  github.com/ethereum/go-ethereum/eth/gasprice
  github.com/ethereum/go-ethereum/eth/tracers
  github.com/ethereum/go-ethereum/eth
  github.com/ethereum/go-ethereum/les
  github.com/ethereum/go-ethereum/ethstats
  github.com/ethereum/go-ethereum/cmd/utils
  github.com/ethereum/go-ethereum/cmd/geth
  ```
  
  `make geth` fails under Windows Subsystem for Linux like this:
```bash
$ make geth
build/env.sh go run build/ci.go install ./cmd/geth
internal/build/azure.go:23:2: cannot find package "github.com/Azure/azure-storage-go" in any of:
/mnt/c/Users/mslin_000/go/src/github.com/ethereum/go-ethereum/build/_workspace/src/github.com/ethereum/go-ethereum/vendor/github.com/Azure/azure-storage-go (vendor tree)
/usr/lib/go-1.10/src/github.com/Azure/azure-storage-go (from $GOROOT)
/mnt/c/Users/mslin_000/go/src/github.com/ethereum/go-ethereum/build/_workspace/src/github.com/Azure/azure-storage-go (from $GOPATH)
Makefile:15: recipe for target 'geth' failed
make: *** [geth] Error 1
```
  
5. No need to do this if debugging from IntelliJ GoLand or IDEA: Build all the tools, but only install `geth` in `$GOPATH/bin`:
  ```bash
  $ go install -v ./cmd/geth
  ```

### Create a New Private Ethereum Network {#create}
1. By default Ethereum stores data in a sub-directory of your home directory named `~/.ethereum`. So that the data for the private blockchain is distinct from the public Ethereum blockchain, we’ll tell `geth` to use the `~/.gowalkthrough` directory for data storage.
  ```bash
  $ geth --datadir ~/.gowalkthrough account new
  Your new account is locked with a password. Please give a password. Do not forget this password.
  Passphrase:
  Repeat passphrase:
  Address: {c063a7c2c2d7364f3ea2b31b2aecd408a376fd43}
    ```
  The account address of the default account for the Ethereum network defined by `~/.gowalkthrough` is stored in the in the `keystore` subdirectory. Here is a bash script that saves the address of the default account in an environment variable called `ACCOUNT`:
  ```bash
  $ export ACCOUNT=$( \
    cat ~/.gowalkthrough/keystore/* | \
    python -c "import sys, json; print json.load(sys.stdin)['address']" \
  )
  $ echo $ACCOUNT
c63c56283afe93fd0094d27890397de08e03ad5a
   ```
   The configuration file in `~/.gowalkthrough/keystore/` has a very long name, for me it was `UTC--2018-06-27T21-35-05.846283500Z--c63c56283afe93fd0094d27890397de08e03ad5a`. It is easiest to refer to that file with `~/.gowalkthrough/keystore/*`, which expands to "all filenames in the `~/.gowalkthrough/keystore/` directory". Since there is only one file in that directory, this shorthand works fine.

2. A genesis block needs to be created that will be used by the initial set of nodes that will participate in the network. The genesis block is configured via a JSON file, which we'll call `~/gowalkthrough.json`. Here is a command line incantation to create that file; note that an initial balance of 42,000,000,000,000,000,000 Wei (equivalent to 42 Ether) is specified for the default account:
  ```bash
  $ echo "{
      \"config\": {
          \"chainId\": 555,
          \"homesteadBlock\": 0,
          \"eip155Block\": 0,
          \"eip158Block\": 0
      },
      \"difficulty\": \"20\",
      \"gasLimit\": \"2100000\",
      \"alloc\": {
          \"$ACCOUNT\":
          { \"balance\": \"42000000000000000000\" }
      }
  }" > ~/gowalkthrough.json
  ```
  This is what my `~/gowalkthrough.json` looks like:
  ```bash
  $   $ cat ~/gowalkthrough.json | python -m json.tool        
  {                                                       
      "alloc": {                                          
          "c63c56283afe93fd0094d27890397de08e03ad5a": {   
              "balance": "42000000000000000000"          
          }                                               
      },                                                  
      "config": {                                         
          "chainId": 555,                                 
          "eip155Block": 0,                               
          "eip158Block": 0,                               
          "homesteadBlock": 0                             
      },                                                  
      "difficulty": "20",                                 
      "gasLimit": "2100000"                               
  }                                                       
  ```

3. Initialize the new Ethereum network genesis block with the following [command line](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options):

  ```bash
  $ geth --datadir ~/.gowalkthrough init ~/gowalkthrough.json
  ```
  Typical output is:
  ```
  INFO [06-27|15:47:52] Maximum peer count                       ETH=0 LES=100 total=12
  INFO [06-27|15:47:52] Allocated cache and file handles         database=/home/mslinn/.gowalkthrough/geth/chaindata cache=16 handles=16
  INFO [06-27|15:47:52] Writing custom genesis block
  INFO [06-27|15:47:52] Persisted trie from memory database      nodes=1 size=149.00B time=121µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
  INFO [06-27|15:47:52] Successfully wrote genesis state         database=chaindata                                  hash=34d0fe…3093f4
  INFO [06-27|15:47:52] Allocated cache and file handles         database=/home/mslinn/.gowalkthrough/geth/lightchaindata cache=16 handles=16
  INFO [06-27|15:47:52] Writing custom genesis block
  INFO [06-27|15:47:52] Persisted trie from memory database      nodes=1 size=149.00B time=159µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
  INFO [06-27|15:47:52] Successfully wrote genesis state         database=lightchaindata                                  hash=34d0fe…3093f4
  ```

### Start the New Ethereum Network  {#start}
You can run the new Ethereum network several ways:

1. Run the network without debugging
  a. Let it run freely
  b. Attach to the process remotely.

2. Under the control of a debugger, which can be a bit complicated, but is necessary in order to set breakpoints and follow along with the walkthrough. 

The first option is the simplest: freely run the network without debugging. Let's explore that option first.

#### Start Without Debugging Enabled {#nodebug}
1. These instructions do not launch a test network, instead, they launch a private, live, full Ethereum network:
  ```bash
  $ geth --syncmode full --cache 64 --maxpeers 12 \
    --rpcapi --wsapi \
    --datadir ~/.gowalkthrough
  ```
  The `--rcapi` option above causes `geth` to create a file called `~/.gowalkthrough/geth.ipc`; that file is automatically deleted when `geth` stops.
    
  Typical output looks like:
  ```
  INFO [06-27|16:25:39] Maximum peer count                       ETH=0 LES=100 total=12
  INFO [06-27|16:25:39] Starting peer-to-peer node               instance=Geth/v1.8.11-stable-dea1ce05/linux-amd64/go1.10
  INFO [06-27|16:25:39] Allocated cache and file handles         database=/home/mslinn/.gowalkthrough/geth/lightchaindata cache=48 handles=512
  INFO [06-27|16:25:39] Initialised chain configuration          config="{ChainID: 555 Homestead: 0 DAO: <nil> DAOSupport: false EIP150: <nil> EIP155: 0 EIP158: 0 Byzantium: <nil> Constantinople: <nil> Engine: unknown}"
  INFO [06-27|16:25:39] Disk storage enabled for ethash caches   dir=/home/mslinn/.gowalkthrough/geth/ethash count=3
  INFO [06-27|16:25:39] Disk storage enabled for ethash DAGs     dir=/home/mslinn/.ethash                    count=2
  INFO [06-27|16:25:39] Loaded most recent local header          number=0 hash=1749e0…53d783 td=20
  INFO [06-27|16:25:39] Starting P2P networking
  INFO [06-27|16:25:41] UDP listener up                          net=enode://df1b9c450f2e868a5fc148dbeaa4e2bca5931aadc794878090c6df1c89d53ad1ba4ad7ac58b80e5981e47dd58a093873bf24b32ced60cb5e0cf992f791130ac8@[::]:30303
  WARN [06-27|16:25:41] Light client mode is an experimental feature
  INFO [06-27|16:25:41] RLPx listener up                         self="enode://df1b9c450f2e868a5fc148dbeaa4e2bca5931aadc794878090c6df1c89d53ad1ba4ad7ac58b80e5981e47dd58a093873bf24b32ced60cb5e0cf992f791130ac8@[::]:30303?discport=0"
  INFO [06-27|16:25:41] IPC endpoint opened                      url=/home/mslinn/.gowalkthrough/geth.ipc
    ```
  
  About 3 messages similar to the last one should appear every second.

#### Debug with GoLand / IntelliJ {#intellij}
[StackOverflow](https://ethereum.stackexchange.com/a/27649/25828) has a posting that discusses this.

> First make sure you have the latest version, EAP 15, 173.2696.28, and you are using the latest version of Go, 1.9, as that's preferred for a better debugging experience due to the recent improvements in Go with regards to debugging.

> Then, go to **Run** | **Edit Configurations** | **Go Applications** | select the run configuration you want to edit | **Run kind** and change it to `File` from `Package`. Then type the name of the package, for example, `github.com/ethereum/go-ethereum/cmd/geth` and save the settings. Then go to **Run** | **Debug...** and select the run configuration you've edited previously and select that as a debug run.

> I've also created a small video which should guide you on how to change the **Run kind** for a **Run Configuration**, [you can see it here](https://www.youtube.com/watch?v=ko-wKntCLjg).

Be sure to specify the same command-line options as in the previous section: `--syncmode full --cache 64 --maxpeers 12 --rpcapi --wsapi --datadir ~/.gowalkthrough`.

** Fails: **
```
C:\Go\bin\go.exe build -i -o C:\Users\mslin_000\AppData\Local\Temp\___geth_server.exe github.com/ethereum/go-ethereum/cmd/geth #gosetup
C:\Users\mslin_000\go\src\github.com\ethereum\go-ethereum\node\node.go:36:2: cannot find package "github.com/prometheus/prometheus/util/flock" in any of:
	C:\Users\mslin_000\go\src\github.com\ethereum\go-ethereum\vendor\github.com\prometheus\prometheus\util\flock (vendor tree)
	C:\Go\src\github.com\prometheus\prometheus\util\flock (from $GOROOT)
	C:\Users\mslin_000\go\src\github.com\prometheus\prometheus\util\flock (from $GOPATH)
```

### Attach to a Running `geth` Process {#gethAttach}
This is useful to see what `geth` does in response to various inputs. First, a refresher on the Linux utilities used:

`man pgrep` describes a useful tool for discovering process IDs.

```
NAME
       pgrep, pkill - look up or signal processes based on name and other attributes

SYNOPSIS
       pgrep [options] pattern
       pkill [options] pattern

DESCRIPTION
       pgrep looks through the currently running processes and lists the process IDs which match the selection criteria to stdout.  All the criteria have to match.  For example,

              $ pgrep -u root sshd

       will only list the processes called sshd AND owned by root.  On the other hand,

              $ pgrep -u root,daemon

       will list the processes owned by root OR daemon.

       pkill will send the specified signal (by default SIGTERM) to each process instead of listing them on stdout.
```       

`strace -fp <pid>` connects to all existing threads

The `strace` command's `-p` option accepts a comma-separated list of pids. The following incantation uses `pgrep` and `paste` to create that list, and captures system calls from all threads within the process.

```bash
sudo strace -t -p $(ls /proc/$(pgrep geth)/task -1 | paste -sd "," -) \
  -o geth.strace
```

The file `geth.strace` will contain something like:
```
strace: Process 23019 attached
strace: Process 23020 attached
strace: Process 23021 attached
strace: Process 23022 attached
strace: Process 23023 attached
strace: Process 23024 attached
strace: Process 23025 attached
strace: Process 23026 attached
strace: Process 23027 attached
strace: Process 23028 attached
strace: Process 23029 attached
strace: Process 23030 attached
strace: Process 23031 attached
strace: Process 23032 attached
strace: Process 23033 attached
```

### Attach a JavaScript Console {#js}
3. In another terminal console, start a JavaScript console that connects to the above running `geth` instance with this incantation:
  
  ```bash
  $ geth attach ~/.gowalkthrough/geth.ipc
  Welcome to the Geth JavaScript console!

  instance: Geth/v1.8.11-stable-dea1ce05/linux-amd64/go1.10
  coinbase: 0xd1c037b9d67b8b0af6003cfaa6951adfa1f67d89
  at block: 0 (Wed, 31 Dec 1969 16:00:00 STD)
   datadir: /home/mslinn/.gowalkthrough
   modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
  
  >
  ```

4. Run the following JavaScript commands to create the default account; notice its balance is 42 Ether:

  ```javascript
  > personal.newAccount()
  Passphrase:
  Repeat passphrase:
  "0xbc2dba1e18dd874707c4e495b139138d050a5846"
  
  > web3.fromWei(eth.getBalance(eth.coinbase), "ether")
  42
  
  > TODO write me
  ```
  
### Shut Down  {#stop}
1. Press Ctrl-C in the first terminal console to terminate `geth`. Output generated by `geth` as it shuts down will look something like:
  ```
  INFO [06-27|13:39:03] IPC endpoint closed                      endpoint=/home/mslinn/.gowalkthrough/geth.ipc
  INFO [06-27|13:39:03] Writing cached state to disk             block=46376 hash=7efacc…f41d6c root=d4e03e…041271
  INFO [06-27|13:39:03] Persisted trie from memory database      nodes=283 size=49.29kB time=9.335ms gcnodes=1617 gcsize=407.84kB gctime=77.473ms livenodes=1 livesize=0.00B
  INFO [06-27|13:39:03] Writing cached state to disk             block=46375 hash=83aa1c…be0d26 root=d4e03e…041271
  INFO [06-27|13:39:03] Persisted trie from memory database      nodes=0   size=0.00B   time=24µs    gcnodes=0    gcsize=0.00B    gctime=0s       livenodes=1 livesize=0.00B
  INFO [06-27|13:39:03] Writing cached state to disk             block=46249 hash=b4cdf5…f24d50 root=d4e03e…041271
  INFO [06-27|13:39:03] Persisted trie from memory database      nodes=0   size=0.00B   time=9µs     gcnodes=0    gcsize=0.00B    gctime=0s       livenodes=1 livesize=0.00B
  INFO [06-27|13:39:03] Blockchain manager stopped
  INFO [06-27|13:39:03] Stopping Ethereum protocol
  WARN [06-27|13:39:03] Synchronisation failed, retrying         err="header processing canceled (requested)"
  INFO [06-27|13:39:03] Ethereum protocol stopped
  INFO [06-27|13:39:03] Transaction pool stopped
  INFO [06-27|13:39:03] Database closed                          database=/home/mslinn/.gowalkthrough/geth/chaindata
  ```

2. Press Ctrl-D in the JavaScript console to exit.
