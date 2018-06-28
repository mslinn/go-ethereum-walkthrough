# Setup {#setup}

These instructions set up a new private, live Ethereum network with pre-allocated funds in the default account. Because this is a new network, with no previous transactions to load, it boots up immediately. The balance in the default account for this network will be initialized when the genesis block is created. This is the easiest way to set up an account with a non-zero balance.

## Check out `go-ethereum` {#checkout}
Ensure that [`GOPATH`](https://github.com/golang/go/wiki/GOPATH) is set:
```bash
$ echo $GOPATH
/home/mslinn/go
```
Create the directory to hold `go-ethereum`. It must be exactly this name:

```bash
$ mkdir -p $GOPATH/src/github.com/ethereum

$ cd $GOPATH/src/github.com/ethereum/
```

Check out `go-ethereum`:
```bash
$ git clone git@github.com:ethereum/go-ethereum.git go-ethereum

$ cd go-ethereum
```

Use [`govendor`](https://github.com/kardianos/govendor) to fetch all dependencies specified in the `vendor` directory except `azure-storage-go` (which is broken), then recreate all generated code:

```bash
$ go get -u github.com/kardianos/govendor

$ govendor remove github.com/Azure/azure-storage-go

$ govendor fetch +v

$ govendor generate +l
```

Build all the tools, but only install `geth` in `$GOPATH/bin`:
```bash
$ go install -v ./cmd/geth
```

## Create a New Private Ethereum Network {#create}
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

## Start the New Ethereum Network  {#start}
TODO: I think the following is probably wrong; the light protocol seems inappropriate for what we need to demonstrate. Also, this happens in the JavaScript console only when light mode:
```
> eth.coinbase
Error: not supported
    at web3.js:3143:20
    at web3.js:6347:15
    at get (web3.js:6247:38)
    at <unknown>
```

The `geth` node uses the [light client protocol](https://github.com/ethereum/wiki/wiki/Light-client-protocol) for this walkthrough. Ethereum&apos;s light client protocol allows for small devices such as the [Raspberry Pi](https://www.rs-online.com/designspark/exploring-ethereum-with-raspberry-pi-part-1-getting-started) to join the network, download block headers as they appear, and only validate certain pieces of state on-demand as required by their users.

4. These instructions do not launch a test network, instead, they launch a private, live, full Ethereum network:
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
  
## Attach a JavaScript Console {#js}
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
  
## Shut Down  {#stop}
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
