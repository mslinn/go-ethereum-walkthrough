## Overview

_This is a work in progress. Comments and suggestions are welcome._

This is a walkthrough of some of the core source files in the `master` branch of the official [Go language](https://golang.org/) implementation of Ethereum \([`go-ethereum`](https://github.com/ethereum/go-ethereum)\), which includes the [`geth`](https://github.com/ethereum/go-ethereum/tree/master/core/vm) command-line Ethereum client program, along with many others. Ethereum clients include an implementation of the Ethereum Virtual Machine \(EVM\), which are able to parse and verify the Ethereum blockchain, including smart contracts, and provides interfaces to create transactions and mine blocks.

This document is up to date with the upcoming `go-ethereum` v1.8.12.

I've added some suggestions for how the source code might be improved \(to locate them, search for `Suggestion`\). If there is general agreement that these suggestions make sense \(tell me in the comments!\) then I'll create a pull request. Anything I don't know or am unsure about is marked with `TODO`.

