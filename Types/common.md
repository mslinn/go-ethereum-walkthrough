# Common Types {#common}
Foundational types used throughout `go-ethereum`.

Two types are defined in [`common/types.go`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go): `Address` and `Hash`.

## `Address` {#address}
[`Address`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go#L137-L138) is defined as an array of 20 `byte`s:

```go
const (
    HashLength    = 32
    AddressLength = 20
)

// Address represents the 20 byte address of an Ethereum account.
type Address [AddressLength]byte
```

## `Hash` {#hash}
[`Hash`](https://github.com/ethereum/go-ethereum/blob/master/common/types.go#L43) is defined as an array of 32 `byte`s:

```go
// Hash represents the 32 byte Keccak256 hash of arbitrary data.
type Hash [HashLength]byte
```

