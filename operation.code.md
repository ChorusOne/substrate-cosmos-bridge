``` 
# Every function here represents operation possible on this substrate ibc client

# takes blocks and routes through babe and grandpa
# if everything goes well, it will be stored inside IBCDB
# import queue is also responsible for pruning db time to time
# So, no separate function is required

# Error here could typically mean we need to mark misbehaviour 
function consume_blocks(import_queue, storage, blocks) returns OptionalError
  var was_import_successful = import_queue.import_blocks(blocks)
  if not was_import_successful {
     return Error
  }
  return Nothing
end


```