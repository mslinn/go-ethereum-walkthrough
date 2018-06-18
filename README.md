## Overview

_This is a work in progress. Comments and suggestions are welcome._

This is a walkthrough of some of the core source files in the `master` branch of the official [Go language](https://golang.org/) implementation of Ethereum \([`go-ethereum`](https://github.com/ethereum/go-ethereum)\), which includes the [`geth`](https://github.com/ethereum/go-ethereum/tree/master/core/vm) command-line Ethereum client program, along with many others. Ethereum clients include an implementation of the Ethereum Virtual Machine \(EVM\), which are able to parse and verify the Ethereum blockchain, including smart contracts, and provides interfaces to create transactions and mine blocks.

This document is up to date with the upcoming `go-ethereum` v1.8.12.

I've added special comments in the displayed source code, in this format:
```go
var x = 41
var y = 42   // <<=== #1
var z = 43
```
The above comment indicates that the line containing the definition of a variable called `y` is the first subject of discussion; the other lines merely provide context.

If more than one line is referenced, the referenced block of code is demarked with `start` and `end` comments. In the following example, the first line is reference #5, while the entire `Run` function is reference #6:
```go
manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol{ // <<=== #5
    Name: ProtocolName,
    Version: version,
    Length: ProtocolLengths[i],
    Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error { // <<=== #6 start
        peer := manager.newPeer(int(version), p, rw)
        select {
        case manager.newPeerCh &lt;- peer:
            manager.wg.Add(1)
            defer manager.wg.Done()
            return manager.handle(peer)</b>
        case &lt;-manager.quitSync:
            return p2p.DiscQuitting
        }
    }, // <<=== #6 end
    NodeInfo: func() interface{} {
    return manager.NodeInfo()
},
```

I've added some suggestions for how the source code might be improved \(to locate them, search for `Suggestion`\). If there is general agreement that these suggestions make sense \(tell me in the comments!\) then I'll create a pull request. Anything I don't know or am unsure about is marked with `TODO`.

