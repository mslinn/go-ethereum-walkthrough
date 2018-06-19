# Referenced Types {#types}
In addition to the [common types](common.md#common) used throughout `go-ethereum`, the following types are important for smart contracts.

## `operation` {#operation}
The [`operation`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/jump_table.go#L35-L51) `struct` defines an EVM opcode:

```go
type operation struct {
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
}
```

Notice the `jumps` property, a Boolean, which if set indicates that the program counter should not increment after executing any form of jump opcode.

The `destinations` type maps the hash of a smart contract to a bit vector for each the smart contract&rsquo;s entry points. If a bit is set, that indicates the EMV&rsquo;s program counter should increment after executing the entry point. [`analysis.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/analysis.go#L25-L28) defines the `destinations` type like this:

```go
// destinations stores one map per contract (keyed by hash of code).
// The maps contain an entry for each location of a JUMPDEST instruction.
type destinations map[common.Hash]bitvec
```

## `InstructionSet` {#instruction_set}
The EVM instruction set for each version of the EVM are defined in [`jump_table.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/jump_table.go#L60-L951). 
