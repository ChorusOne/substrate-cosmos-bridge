```
# custom database implementation that reads and write to data blob that 
# will be acting as IBC's opaque client data
# We are not assuming what actual format of data_blob is, 
# but at least it need to support storing key value pair and iteration

class IBCDB which implements Database interface {
  static func new(data_blob) {
    return IBCDB { data_blob = data_blob }
  }

  func get(key) returns OptinalValue {
    return data_blob.get_key(key)
  }

  func get_by_prefix(prefix) returns OptionalKeyArray {
    var matching_keys OptionalKeyArray
    data_blob.iterate(func (key) {
      if prefix.IsPrefixOf(key) {
        matching_keys.add(key)
      }
    })
    return matching_keys
  }

  func write_buffered(transaction) {
    data_blob.write(transaction)
  }
  
  func iter() returns Iterator {
    return data_blob.iterator()
  }
}

```