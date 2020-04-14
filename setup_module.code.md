```
# Implementation of substrate light client, uses backend to provide various apis related to blockchain execution, fact  checking etc.
global var client;

# Implementation of substrate light Backend interface, mainly provides shared access to blockchain
global var backend;

# Implementation of substrate light Blockchain interface, uses storage to provide api to access blockchain data
global var blockchain;

# This need to be custom implemented
# storage is used by blockchain object as underlying storage of blockchain data
# Typically this is mainly leveldb, but in our case this need to operate on a data blob to make it wasm compatible
global var storage;

# import queue is object that is going to consume incoming blocks and finality proof
# validate them using block_import and finality_proof_importer
# if it is valid, it will be stored using client object's api in blockchain's storage
function setup_import_queue(block_import, grandpa_block_import) returns import_queue
  var import_queue = call external constructor import_queue(block_import, grandpa_block_import as FinalityProofImport)
  return import_queue
end

# grandpa_block_import is object with responsibility of validating finality
function setup_grandpa_block_import(fetch_checker) returns babe_block_import
  var grandpa_block_import = call external constructor grandpa_block_import(fetch_checker)
  return grandpa_block_import
end

# We need to wrap grandpa_block_import into babe_block_import which enables babe_block_import to 
# push babe verified block to grandpa_block_import object
function setup_babe_block_import(grandpa_block_import) returns babe_block_import
  var babe_block_import = call external constructor babe_block_import(grandpa_block_import)
  return babe_block_import
end

# fetch checker is useful to prove blockchain header we received is valid
function setup_fetch_checker() returns fetch_checker
  var light_data_checker = call external constructor light_data_checker()
  return light_data_checker cast as fetch_checker
end

# Sets up import_queue, client and backend
# If contract caller have some way to snapshot current state, this only need to be done once. 
# otherwise this need to be done before any operation code can be executed
function setup returns import_queue
  initialize storage using IBCDB
  initialize blockchain using storage
  initialize backend using blockchain
  initialize client using backend
  
  let fetch_checker = setup_fetch_checker()
  let grandpa_block_import = setup_grandpa_block_import(fetch_checker)
  let babe_block_import = setup_babe_block_import(grandpa_block_import)
  let import_queue = setup_import_queue(babe_block_import, grandpa_block_import)
  return import_queue
end
```