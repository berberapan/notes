Slices are similar to ArrayLists in other languages and are in a practical sense dynamic arrays. Whilst arrays have a set size you can add to the size of a slice. The slices use underlying arrays and automatically creates new space in contiguous memory if size is bigger than the underlying array.  

##### Array

Slices are used over arrays in most cases but it's good to know how to declare both.

```go
// Declaring an array of 5 ints
var nums [5]int

// Declaring with initialisation
oddNums := [5]int{1, 3, 5, 7, 9}
```

If making a slice from an array you just use the indices. Like in many languages the low index point is inclusive and the high point exclusive.

```go
sliceOdd := oddNums[1:3]
// sliceOdd = {3, 5}

sliceOdd := oddNums[3:]
// sliceOdd = {7, 9}

sliceOdd := oddNums[:2]
// sliceOdd = {1, 3}

sliceOdd := oddNums[:]
// sliceOdd = {1, 3, 5, 7, 9}
```

##### Make

You can use the make function to create new slices, that gives you the option to set the length and capacity. 

`func make([]T, len, cap) []T`

```go
example := make([]int, 5, 10)
```


Capacity sets the number of elements in the underlying array of the slice. The maximum length of the slice before reallocation is necessary. Unless you're doing optimisation you don't touch this as slices will grow automatically so it can be omitted from function.

Length is just the current number of elements it contains. Say your function takes in a slice of unknown size that you want to process in some way you can just use the make function with len() for the slice you're taking.

If you want to create a slice with a specific set of values you can just use a slice literal. The difference with the array literal is that you leave the brackets empty.

```go
example := []int{1, 2, 3}
```

##### Variadic

Many functions can take a arbitrary number of arguments. If "..." is in the function signature it's a variadic function and the arguments are collected as slice.

```go
func concat(strs ...string) string {
    final := ""
    // strs is just a slice of strings
    for i := 0; i < len(strs); i++ {
        final += strs[i]
    }
    return final
}

func main() {
    final := concat("Hello ", "there ", "friend!")
    fmt.Println(final)
    // Output: Hello there friend!
}
```

If you want to pass a slice into a variadic function you use the spread operator which is "..." after the slice in the function call.

```go
func printStrings(strings ...string) {
	for i := 0; i < len(strings); i++ {
		fmt.Println(strings[i])
	}
}

func main() {
    names := []string{"bob", "sue", "alice"}
    printStrings(names...)
}
```

##### Append#

If you want to add to your slice there's a built in append function.

`func append(slice []Type, elems ...Type) []Type`

As append is a variadic you can append in all these ways

```go
slice = append(slice, oneThing)
slice = append(slice, firstThing, secondThing)
slice = append(slice, anotherSlice...)
```

Best practice for append is to only append to the slice itself. Append function creates a new underlying array only if there isn't any capacity left which can create bugs when values are not what you expect.
##### Two-Dimensional slices

If you need a matrix you can use a slice that holds other slices.

```go
matrix := [][]int{}

// example of 2D slice where each column is value of row * column index

[0 0 0 0 0 0 0 0 0 0]
[0 1 2 3 4 5 6 7 8 9]
[0 2 4 6 8 10 12 14 16 18]
[0 3 6 9 12 15 18 21 24 27]
[0 4 8 12 16 20 24 28 32 36]
[0 5 10 15 20 25 30 35 40 45]
[0 6 12 18 24 30 36 42 48 54]
[0 7 14 21 28 35 42 49 56 63]
[0 8 16 24 32 40 48 56 64 72]
[0 9 18 27 36 45 54 63 72 81]
```

#### Modify values

If you want to change the value on the original slice in a loop for example, use the index. Otherwise if you use the value it will just be a copy.

```go
// Using index - modifies original
for i := range messages {
    messages[i].field = newValue // Changes actual element
}

// Using value - works with copy
for _, msg := range messages {
    msg.field = newValue // Changes only the copy
}
```