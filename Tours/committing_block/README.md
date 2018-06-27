# Tour: Committing a Block {#committing-block}

_This tour follows a block in the `geth` client from when it arrives at a network socket until it is committed to the blockchain._

## Setup

We will set up a new test network with preallocated funds in the default account.

The `geth` client uses the [light client protocol](https://github.com/ethereum/wiki/wiki/Light-client-protocol) for this walkthrough. Ethereum&apos;s light client protocol allows for small devices such as the [Raspberry Pi](https://www.rs-online.com/designspark/exploring-ethereum-with-raspberry-pi-part-1-getting-started) to join the network, download block headers as they appear, and only validate certain pieces of state on-demand as required by their users.

1. A name is required for the new blockchain network and for the purposes of this tour, we’ll use `GoWalkthrough`. By default Ethereum stores data in a sub-directory of your home directory named `.ethereum`, which is a hidden directory on Linux/BSD/Darwin. So the data for the private blockchain is distinct from the public Ethereum blockchain, we’ll use `~/.gowalkthrough`.
  ```bash
  $ geth --datadir ~/.gowalkthrough account new
  Your new account is locked with a password. Please give a password. Do not forget this password.
  Passphrase:
  Repeat passphrase:
  Address: {c063a7c2c2d7364f3ea2b31b2aecd408a376fd43}
    ```
  The account address needs to be saved, so that the new Ethereum network can be initialized with preallocated funds. Here is a bash script that saves the address in an environment variable called `ACCOUNT`:
  ```bash
  $ export ACCOUNT=$( \
    cat ~/.gowalkthrough/keystore/* | \
    python -c "import sys, json; print json.load(sys.stdin)['address']" \
  )
  $ echo $ACCOUNT
c63c56283afe93fd0094d27890397de08e03ad5a
   ```
   The configuration file in `~/.gowalkthrough/keystore/` has a very long name, for me it was `UTC--2018-06-27T21-35-05.846283500Z--c63c56283afe93fd0094d27890397de08e03ad5a`. It is easiest to refer to that file with `~/.gowalkthrough/keystore/*`, which expands to "all filenames in the `~/.gowalkthrough/keystore/` directory". Since there is only one file in that directory, this shorthand works fine.

2. A genesis block needs to be created that will be used by the initial set of nodes that will participate in the network. The genesis block is configured via a JSON file, which we'll call `~/.gowalkthrough.json`. Here is an easy way to create that file; note that an initial balance of 420,000,000,000,000,000,000 Wei is specified for the default account:
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
          { \"balance\": \"420000000000000000000\" }
      }
  }" > ~/.gowalkthrough.json
  ```
  This is what my file looks like:
  ```bash
  $ cat .gowalkthrough/keystore/* | python -m json.tool
{
    "address": "c63c56283afe93fd0094d27890397de08e03ad5a",
    "crypto": {
        "cipher": "aes-128-ctr",
        "cipherparams": {
            "iv": "420862fb4d32861b5f8bc8ee5dec9de2"
        },
        "ciphertext": "e2de36151b3ec2af0655f3c996d7b295265b7bce84f855ea1a22a85a30ba011c",
        "kdf": "scrypt",
        "kdfparams": {
            "dklen": 32,
            "n": 262144,
            "p": 1,
            "r": 8,
            "salt": "fa16c26b1afcdbd61b4230768fa92c691c4ce9bc192d3636207d6f59ec34d625"
        },
        "mac": "08fd026accc839b892da18d2b2a7d4ca2cdd49ace715057802be18c65ca7cd1f"
    },
    "id": "c570d079-db1c-4b70-907c-56b0306d7a47",
    "version": 3
}
  ```

2. The [command line](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options) to set up for this tour is:

  ```bash
  $ geth --syncmode light --cache 64 --maxpeers 12 \
    --rpcapi --wsapi --rinkeby \
    --datadir ~.gowalkghrough init ~gowalkghrough.json
  ```
  
  Typical output looks like:
  ```
  INFO [06-27|13:22:37] Maximum peer count                       ETH=0 LES=100 total=12
  INFO [06-27|13:22:37] Starting peer-to-peer node               instance=Geth/v1.8.11-stable-dea1ce05/linux-amd64/go1.10
  INFO [06-27|13:22:37] Allocated cache and file handles         database=/home/mslinn/.ethereum/rinkeby/geth/lightchaindata cache=48 handles=512
  INFO [06-27|13:22:38] Persisted trie from memory database      nodes=355 size=51.91kB time=430µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
  INFO [06-27|13:22:38] Initialised chain configuration          config="{ChainID: 4 Homestead: 1 DAO: <nil> DAOSupport: true EIP150: 2 EIP155: 3 EIP158: 3 Byzantium: 1035301 Constantinople: <nil> Engine: clique}"
  INFO [06-27|13:22:38] Loaded most recent local header          number=83648 hash=074e9f…47194d td=164879
  INFO [06-27|13:22:38] Starting P2P networking
  INFO [06-27|13:22:40] UDP listener up                          net=enode://fe9c3c8f29c364be0fc7659e1b6f8e9eb90c8ed7861d0b02474d3b46adff1581cc6f4be61a8769da087449358c24a2642cebd3a6797ce49bdeeca087ceea836c@[::]:30303
  WARN [06-27|13:22:40] Light client mode is an experimental feature
  INFO [06-27|13:22:40] RLPx listener up                         self="enode://fe9c3c8f29c364be0fc7659e1b6f8e9eb90c8ed7861d0b02474d3b46adff1581cc6f4be61a8769da087449358c24a2642cebd3a6797ce49bdeeca087ceea836c@[::]:30303?discport=0"
  INFO [06-27|13:22:40] IPC endpoint opened                      url=/home/mslinn/.ethereum/rinkeby/geth.ipc
  INFO [06-27|13:22:41] Block synchronisation started
  INFO [06-27|13:22:43] Imported new block headers               count=0 elapsed=225.794ms number=83840 hash=5e83e3…65964c ignored=192
  ```
  
  About 3 messages similar to the last one should appear every second.
  
3. In another terminal console, start a JavaScript console that connects to the above running `geth` instance with this incantation:
  
  ```bash
  $ geth attach ~/.ethereum/rinkeby/geth.ipc
  ```

4. Run the following JavaScript commands to create a transaction:
```javascript
> personal.newAccount()
> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
> TODO write me
```
  
5. Press CTRL-C from the first terminal console to terminate `geth`. Output generated by `geth` as it shuts down will look something like:
  ```
  INFO [06-27|13:39:03] IPC endpoint closed                      endpoint=/home/mslinn/.ethereum/rinkeby/geth.ipc
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
  INFO [06-27|13:39:03] Database closed                          database=/home/mslinn/.ethereum/rinkeby/geth/chaindata
  ```
  
## Attribution {#attribution}
This is a modified version of [`go-ethereum-code-walkthrough.md`](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [`gsalgado`](https://github.com/gsalgado). Code references were updated, more detail was added, information about types was added, a redundant step was removed, spelling was corrected and code snippets were added.

## Preconditions {#preconditions}
`TODO` add the missing steps for the following into the initialization subtour below:
* Program is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected to a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## [Initialization](initialization.md#initialization)

## [Message Handling](handling.md#handling)
