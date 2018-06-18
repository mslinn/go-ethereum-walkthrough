### `contracts.go` {#contracts-go}

This file is responsible for executing smart contracts on the EVM.

#### Imports {#imports}

The following imports are used:

* [Package `sha256`](https://golang.org/pkg/crypto/sha256/) from the `crypto` project implements the SHA224 and SHA256 hash algorithms as defined in FIPS 180-4.
* [`errors`](https://godoc.org/github.com/pkg/errors), the Go language simple error handling primitives, such as `error`.
* [`math/big`](https://golang.org/pkg/math/big/) implements arbitrary-precision arithmetic \(big numbers\).
* Other packages in this project \(`go-ethereum`\):

  ```go
  "github.com/ethereum/go-ethereum/common"
  "github.com/ethereum/go-ethereum/common/math"
  "github.com/ethereum/go-ethereum/crypto"
  "github.com/ethereum/go-ethereum/crypto/bn256"
  "github.com/ethereum/go-ethereum/params"
  ```

  `Suggestion`: Again, I think the above imports would have been better specified as relative imports:

  ```go
  "../../common"
  "../../common/math"
  "../../crypto"
  "../../crypto/bn256"
  "../../params"
  ```

* [`ripemd160`](https://godoc.org/golang.org/x/crypto/ripemd160) implements the [RIPEMD-160 hash algorithm](http://homes.esat.kuleuven.be/~bosselae/ripemd160.html), a secure replacement for the MD4 and MD5 hash functions. These hashes are also termed RIPE message digests.

#### Type `PrecompiledContract` {#type-precompiledcontract}

[`PrecompiledContract`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contracts.go#L32-L38) is the interface for native Go smart contracts. This interface is used by precompiled contracts, as we will see next. `Contract` is a `struct` defined in [`contract.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contract.go).

#### Pre-Compiled Contract Maps {#pre-compiled-contract-maps}

These maps specify various types of cryptographic hashes and utility functions, accessed via their address.

[`PrecompiledContractsHomestead`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contracts.go#L40-L47) contains the default set of pre-compiled contract addresses used in the Frontier and Homestead releases of Ethereum: `ecrecover`, `sha256hash`, `ripemd160hash` and `dataCopy`.

[`PrecompiledContractsByzantium`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contracts.go#L49-L60) contains the default set of pre-compiled contract addresses used in the Byzantium Ethereum release. All of the previously defined pre-compiled contract addresses are provided in Byzantium, plus: `bigModExp`, `bn256Add`, `bn256ScalarMul` and `bn256Pairing`.

`Suggestion`: Some code is duplicated, whereby the contents of `PrecompiledContractsHomestead` are incorporated into `PrecompiledContractsByzantium` by listing the values again; this would be better expressed by referencing the values of `PrecompiledContractsHomestead` instead of duplicating them.

#### Contract Evaluator Function {#contract-evaluator-function}

The `RunPrecompiledContract` function runs and evaluates the output of a precompiled contract. It accepts three parameters:

* A `PrecompiledContract` instance.
* A byte array of input data.
* A reference to a `Contract`, defined in [`contract.go`](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contract.go#L44-L65), discussed above.

The function returns:

* A byte array containing the output of the contract.
* An `error` value, which could be `nil`.

#### Other Functions {#other-functions}

* `RunPrecompiledContract` – runs and evaluates the output of a precompiled contract; returns the output as a byte array and an `error`.
* `RequiredGas` \(overloaded\) – Computes the gas required for input data, specified as a byte array and returns a `uint64`.
* `Run` \(overloaded\) – Computes the smart contract for input data, specified as a byte array and returns the result as a left-padded byte array and an `error`.
* `newCurvePoint` – Unmarshals a binary blob into a bn256 elliptic curve point. BN-curves are an elliptic curves suitable for cryptographic pairings that provide high security and efficiency cryptographic schemes. See the IETF paper on [Barreto-Naehrig Curves](https://tools.ietf.org/id/draft-kasamatsu-bncurves-01.html) for more information.



