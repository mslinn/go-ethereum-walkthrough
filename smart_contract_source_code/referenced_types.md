### Referenced Types {#referenced-types}

Two of the types used in the source files that we would like to understand are defined in [`common/types.go`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go). Let&#039;s look at them first.

[`Address`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go#L137-L138) is defined as an array of 20 `byte`s:

<pre>const (
    HashLength    = 32
    AddressLength = 20
)

// Address represents the 20 byte address of an Ethereum account.
type Address [AddressLength]byte</pre>

[`Hash`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go#L43) is defined as an array of 32 `byte`s:

<pre>// Hash represents the 32 byte Keccak256 hash of arbitrary data.
type Hash [HashLength]byte</pre>

The opcodes for each version of the EVM are defined in [`jump_table.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/jump_table.go). The [`operation`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/jump_table.go#L35-L51) `struct` defines the properties:

<pre>type operation struct {
    // execute is the operation function
    execute executionFunc
    // gasCost is the gas function and returns the gas required for execution
    gasCost gasFunc
    // validateStack validates the stack (size) for the operation
    validateStack stackValidationFunc
    // memorySize returns the memory size required for the operation
    memorySize memorySizeFunc

    halts   bool // indicates whether the operation should halt further execution
    jumps   bool // indicates whether the program counter should not increment
    writes  bool // determines whether this a state modifying operation
    valid   bool // indication whether the retrieved operation is valid and known
    reverts bool // determines whether the operation reverts state (implicitly halts)
    returns bool // determines whether the operations sets the return data content
}</pre>

Notice the `jumps` property, a Boolean, which if set indicates that the program counter should not increment after executing any form of jump opcode.

The `destinations` type maps the hash of a smart contract to a bit vector for each the smart contract&#039;s entry points. If a bit is set, that indicates the EMV&#039;s program counter should increment after executing the entry point. [`analysis.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/analysis.go#L25-L28) defines the `destinations` type like this:

<pre>// destinations stores one map per contract (keyed by hash of code).
// The maps contain an entry for each location of a JUMPDEST instruction.
type destinations map[common.Hash]bitvec</pre>