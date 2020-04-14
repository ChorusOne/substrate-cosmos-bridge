```
The simple relayer opens a websocket connection to the Substrate RPC, and subscribes to finalized headers.
Finalized headers will continue to arrive until the websocket connection is closed.
On receipt of a finalzed header, we must request via the same websocket the block hash. The block hash is required
to request the block. On receipt of the block hash, we can query for the block, also via websocket. We require the
block in order to retrieve the justification data. A subsequent request is made to the state_getStorage method, in
order to retrieve the the autoritySet data. All the data for a given block is then packaged into a data structure,
and wrapped in a Cosmos sdk.Msg, which in turn is wrapped in a StdTx, such that it can be submitted to the Cosmos
mempool. Once in the mempool, it will be routed to the appropriate Cosmos SDK module for unmarhsalling and submission
of the collected data into the wasm bytecode client.

ws = open_websocket(substrate_ws_uri)
ws.subscribe("chain_finalizedHeader")  ## subscribe to finalizedHeaders; this will stream until the socket is closed.
ws.on_message(func() {
    switch msg.type {
      case "finalizedHeader":
        store(header)
        ws.send(chain_blockHash, header.block_number)
      case "blockHash":
        store(blockHash)
        ws.send(chain_block, blockHash)
      case "block":  ## we have to retrieve the entire block in order to retrieve the justification
        store(block)
        ws.send(state_getStorage, blockHash, autoritySet)
      case "storage"
        store(authoritySet)
        createComsosTx(fetch(header), fetch(block.justification), fetch(authoritySet))
        signCosmosTx(relayer_pk)
        submitCosmosTx(cosmos_rpc_uri)
    }
})
```
