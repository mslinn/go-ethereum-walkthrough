## Overview {#overview}

_This is a work in progress. Errors certainly exist. Corrections, comments and suggestions are most welcome._

This documentation contains several tours through the [`go-ethereum`](https://github.com/ethereum/go-ethereum) source code. The tours are contained in the last section of this documentation. The penultimate section discusses the major types used in `go-ethereum`, broken into the subsections so they can be conveniently referenced from each of the tours. 

This documentation is up to date with the upcoming `go-ethereum` v1.8.12 source files in the `master` branch of the official [Go language](https://golang.org/) implementation of Ethereum, which includes the [`geth`](https://github.com/ethereum/go-ethereum/tree/master/core/vm) command-line Ethereum client program, along with many others. Ethereum clients include an implementation of the Ethereum Virtual Machine \(EVM\), which are able to parse and verify the Ethereum blockchain, including smart contracts, and provides interfaces to create transactions and mine blocks.

FYI, the Gitter channel for `go-ethereum` is [a`ethereum/go-ethereum`](https://gitter.im/ethereum/go-ethereum).

## Don&apos;t Try To Use Windows
Don&apos;t attempt to use Windows for building or debugging `go-ethereum`. Even Windows Subsystem for Linux is problematic.

## Code Reference Comments {#references}
I&apos;ve added special comments in the displayed source code, in this format:
```go
var x = 41
var y = 42   // <<=== #1
var z = 43
```
The above comment indicates that the line containing the definition of a variable called `y` is the first subject of discussion; the other lines merely provide context.

When a reference would benefit from showing more code, I try to provide that code in a subordinate item with a corresponding identifier; for example, if references `#3a`, `#3b` and `#3c` all have their own code blocks to clarify the explanations, those blocks would be provided under item `3` as `3(a)`, `3(b)` and `3(c)`.

If a block of code is referenced it is demarked with `start` and `end` comments. In the following example, the first line is reference `#5a`, while the entire `Run` function is reference `#5b`:
```go
manager.SubProtocols = append(manager.SubProtocols, p2p.Protocol{ // <<=== #5a
    Name: ProtocolName,
    Version: version,
    Length: ProtocolLengths[i],
    Run: func(p *p2p.Peer, rw p2p.MsgReadWriter) error { // <<=== #5b start
        peer := manager.newPeer(int(version), p, rw)
        select {
        case manager.newPeerCh &lt;- peer:
            manager.wg.Add(1)
            defer manager.wg.Done()
            return manager.handle(peer)</b>
        case &lt;-manager.quitSync:
            return p2p.DiscQuitting
        }
    }, // <<=== #5b end
    NodeInfo: func() interface{} {
    return manager.NodeInfo()
},
```

## TODO and Suggestion {#comments}
I've added some suggestions for how the source code might be improved \(to locate them, search for `Suggestion`\). If there is general agreement that these suggestions make sense \(tell me in the comments!\) then I'll create a pull request. Anything I don't know or am unsure about is marked with `TODO`.

