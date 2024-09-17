Maps are like JSON or dictionaries in Python. It's a data structure that got key value pairing.

Zero value of a map is nil.

Maps can be created with the make() function or using a literal.

Make
```go
ages := make(map[string]int)
ages["John"] = 37
ages["Mary"] = 24
ages["Mary"] = 21 // overwrites 24
```

Literal
```go
ages = map[string]int{
  "John": 37,
  "Mary": 21,
}
```

##### Mutation

```go
// Insert an element
map[key] = element

// Get an element
element = map[key]

// Delete an element
delete(map, key)

// Check if a key exist
element, ok := map[key]
```

When you check for a key the ok value will be true if it exist and false if not. If the key is not in the map the element variable will get the zero value of the map's element type.

##### Key types

Any type can be used as the value in a map but not all types can be used as the key. For a key type to be acceptable it needs to be one that can be comparable. Types like slices, maps and functions can't be compared with a simple '\==' so can't be used for keys.

Structs can be used as keys though, as shown in example below.

```go
type Key struct {
    Path, Country string
}
hits := make(map[Key]int)
```
