### `contract.go` {#contract-go}

This file defines smart contract behavior.

#### Import Suggestion {#imports}

_This suggestion applies to all of the Go source files in the entire project._ 

I think the following absolute import would have been better specified as a relative import:

```go
"github.com/ethereum/go-ethereum/common"
```

The relative import would look like this instead:

```go
"../../common"
```

If relative imports were used instead of absolute imports that point to the github repo, local changes to the project made by a developer would automatically be picked up. As currently written, absolute imports cause local changes to be ignored, in favor of the version on github. It might take a software developer a while to realize that the reason why their changes are ignored by most of the code base is because absoluate imports were used. It would then be painful to for the developer to modify the affected source files throughout the project such that they used relative imports.

#### Types {#types}

The publicly visible [`AccountRef`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contract.go#L30-L40) type is defined as:

```go
// Account references are used during EVM initialisation and
// it&#039;s primary use is to fetch addresses. Removing this object
// proves difficult because of the cached jump destinations which
// are fetched from the parent contract (i.e. the caller), which
// is a ContractRef.
type AccountRef common.Address
```

The same file defines a type cast from `AccountRef` to `Address`:

```go
// Address casts AccountRef to a Address
func (ar AccountRef) Address() common.Address { return (common.Address)(ar) }
```

The [`ContractRef`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contract.go#L25-L28) interface is used by the `Contract` `struct`, which we&#039;ll see in a moment. This `ContractRef` interface just consists of an `Address`.

```go
// ContractRef is a reference to the contract&#039;s backing object
type ContractRef interface {
    Address() common.Address
}
```

The [`Contract`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contract.go#L42-L65) struct defines the behavior of Ethereum smart contracts, and is central to the topic, so here it is in all its glory:

```go
type Contract struct {
    CallerAddress common.Address
    caller    ContractRef
    self      ContractRef

    jumpdests destinations // result of JUMPDEST analysis.

    Code     []byte
    CodeHash common.Hash
    CodeAddr *common.Address
    Input    []byte

    Gas   uint64
    value *big.Int

    Args []byte

    DelegateCall bool
}
```

### Public members 
`CallerAddress` is the `Address` of the caller.

`Code` is a `byte` slice. We don&rsquo;t yet know if this is the smart contract source code, compiled code, or something else.

`CodeHash` is hash of the `Code`.

`CodeAddr` is a pointer to the `Address` (`TODO` of the code, presumably).

`Gas` is the amount of Ethereum gas allocated by the user for executing this smart contract, stored as an unsigned 64-bit integer.

`Value` is a pointer to a big integer. `TODO` possibly this might be the result of executing the contract?

`Args` is a `byte` slice, `TODO` not sure what it is for.

`DelegateCall` is Boolean value, unclear if this means the smart contract was invoked using [`delegatecall`](http://solidity.readthedocs.io/en/v0.4.24/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries). From the documentation: &quot;This means that a contract can dynamically load code from a different address at runtime. Storage, current address and balance still refer to the calling contract, only the code is taken from the called address. This makes it possible to implement the “library” feature in Solidity: Reusable library code that can be applied to a contract’s storage, e.g. in order to implement a complex data structure.&quot;

### Private members
`caller` and `self` are `ContractRef`s, which as we know are really just `Address`es.

`jumpdests` has type `destinations`, which as we&rsquo;ve already discussed defines if the entry point in the smart contract that need the program counter to be incremented after executing.
