```
Cosmos-SDK module comprises a Keeper - which has a reference to the WASM Virtual Machine,
and manages storage, instantiation and execution of client bytecode - and a MessageHandler
which determines how inbound messages are routed. In this case, inbound payload messages are
passed directly to the bytecode functions for verification.

# Cosmos SDK module keeper, responsible for managing state within the module.

class Keeper {

  private var wasmer

  func storeclientcode(clientbytecode) {
    if validate(bytecode) {
      codeid = increment_counter()
      store(key('code', codeid), bytecode)
      return codeid
    }
  }

  func createclient(clientname, codeid, args) {
    wasmerid = wasmer.initialize(codeid, args)
    store(key('client', clientname), wasmerid)
  }

  func executeclient(clientname, method, payload) {
    wasmerid = fetch(clientname)
    response = wasmer.execute(wasmerid, method, payload)
    store(key('state', clientname), response)    
  }

}

# Cosmos SDK message handler, responsible for marshalling inbound messages to the correct functions.

class MsgHandler {
  func registerRoute() {
    // called on module instantiation to register the inbound message types, such that they are routed to module on arrival.
  }

  func handleMsg(msg) {
    switch (msg.type) {
      case WrappedMessage:
        Keeper.execute(clientname, 'import_queue', msg.data)
      case CreateClient:
        Keeper.createclient(msg.data.clientname, msg.data.codeid, ...)
      case StoreCode:
        Keeper.storeclientcode(msg.data)      
  }
}
```
