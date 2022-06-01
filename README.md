# serde_json_tuple_iter
Serialize any iterator of 2-tuples into a JSON map with serde_json, even if the first element is a struct type.

This allows you to work around the "key must be a string" error when trying to serialize a HashMap with a struct key. The output will be the same as if you manually serialized the struct key to a string.

```
#[derive(Clone, Copy, Serialize, PartialEq, Eq, Hash)]
pub struct Test {
  pub a: i32,
  pub b: i32
}

fn main() {
  let mut map = HashMap::<Test, Test>::new();
  map.insert(Test {a: 3, b: 5}, Test {a: 7, b: 9});

  // Fails with Error: key must be a string
  let serialized = serde_json::to_string(&map);
  match serialized {
    Ok(s) => { println!("{}", s); }
    Err(e) => { println!("Error: {}", e); }
  }

  // Long winded workaround that copies the map
  // Prints {"{\"a\":3,\"b\":5}":{"a":7,"b":9}}
  let string_map: HashMap<String, Test> = map.iter().map(|(k, &v)| (serde_json::to_string(k).unwrap(), v)).collect();
  let serialized = serde_json::to_string(&string_map);
  match serialized {
    Ok(s) => { println!("{}", s); }
    Err(e) => { println!("Error: {}", e); }
  }
  
  // Use this crate's utility function - elements are serialized lazily
  // Same output
  let serialized = serde_json_tuple_iter::map_to_json(&mut map.iter());
  match serialized {
    Ok(s) => { println!("{}", s); }
    Err(e) => { println!("Error: {}", e); }
  }

  

  // Utility function also exists for vec of tuples
  // Same output
  let v = vec![(Test {a: 3, b: 5}, Test {a: 7, b: 9})];
  let serialized = serde_json_tuple_iter::vec_to_json(&mut v.iter());
  match serialized {
    Ok(s) => { println!("{}", s); }
    Err(e) => { println!("Error: {}", e); }
  }
}
```

Output:
```
Error: key must be a string
{"{\"a\":3,\"b\":5}":{"a":7,"b":9}}
{"{\"a\":3,\"b\":5}":{"a":7,"b":9}}
{"{\"a\":3,\"b\":5}":{"a":7,"b":9}}
```
