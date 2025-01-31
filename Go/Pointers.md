A variable is a named location in memory that stores a value. When a value is assigned to  a variable it's stored at that specific location.

```go
x := 42
// "x" is the name of a location in memory. That location is storing the integer value of 42
```

Pointers are variables that stores the memory address of another variable. It's pointing to the location and not the data itself.

Pointers can be defined with **\***

```go
var p *int
```

Generating a pointer to an operand is done with **&**

```go
myString := "hello"
myStringPtr := &myString
```

If you define a pointer as `var p *int` above its zero value is nil. It doesn't point to any address. Empty pointers are called "nil pointers".

If you do it like `myStringPtr := &myString`, you get a pointer to the operand. This is the common way to declare a pointer.

Using **\*** operator you dereference a pointer to get the value of the address.

```go
*myStringPtr = "world"                              // set myString through the pointer
fmt.Printf("value of myString: %s\n", *myStringPtr) // read myString through the pointer
// value of myString: world
```

##### Pass by Reference

One of the most common ways to use pointers in Go is to pass variables by reference. So instead of a function receiving a copy of a value it gets the original variable. That means it can update original variable's value. See the difference in the examples below.

No pointer
```go
func increment(x int) {
    x++
    fmt.Println(x)
}


func main() {
    x := 5
    increment(x)
    fmt.Println(x)
    // 5
}
```

Pointer
```go
func increment(x *int) {
    *x++
    fmt.Println(*x)
}

func main() {
    x := 5
    increment(&x)
    fmt.Println(x)
    // 6
}
```

When a pointer to a struct is passed as a value to a function, you should just access the fields as normal. If you try to use dereference the compiler will fail.

##### Nil Pointers

Always have a plan for how to deal with nil pointers. If you attempt to dereference a nil pointer it will cause a runtime error that crashes the program.