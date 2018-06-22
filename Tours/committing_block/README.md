# Tour: Committing a Block {#committing-block}

_This tour follows a block in the `geth` client from the socket until it is committed to the blockchain._

## [Types](types.md#types)

## Attribution {#attribution}
Following is a modified version of [go-ethereum-code-walkthrough.md](https://gist.github.com/gsalgado/16a67aa51207f87e259a7007a2e8d274) by [gsalgado](https://github.com/gsalgado). Code references were updated, more detail was added, information about types was added, a redundant step was removed, spelling was corrected and code snippets were added.

## Preconditions {#preconditions}
* Program is configured and started.
* All services are registered.
* The discovery protocol discovered some nodes.
* The peermanager successfully connected to a node.
* An encrypted multiplexed session is established.
* We are waiting for ingress data in the peer connection.

## [Initialization](initialization.md#initialization)

## [Message Handling](handling.md#handling)
