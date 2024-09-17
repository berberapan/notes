A basic loop in Go is written like in Java and C.

```go
for INITIAL; CONDITION; AFTER {
  // Loop stuff
}

// A loop printing 1 to 10.
for i := 0; i < 10; i++ {
  fmt.Println(i+1)
}
```

You don't have to have a condition when making the loop. It will just keep going until broken or a value is returned. Make sure there's an exit condition if using this type of loop.

```go
for INITIAL; ; AFTER {
  // Infinite loops
}

// Printing 1 to 10
for i := 0; ; i++ {
  if i >= 10 {
    return
  }
  fmt.Println(i + 1)
}
```

Go doesn't have a standard while loop. But as you can omit sections of the for loop you can make it behave just like a while loop. You just give it a condition and nothing else.

```go
for CONDITION {
  // Do stuff until condition is true
}

// Printing 1 to 10
i := 0
for i < 10 {
  fmt.Println(i+1)
  i++
}
```

##### Continue and Break

If you want to stop the current iteration of a loop and go to the next you can use the keyword <span style="color: orange; font-weight: bold">continue</span>.

```go
for i := 0; i < 10; i++ {
  if i % 2 == 0 {
    continue
  }
  fmt.Println(i)
}
// 1
// 3
// 5
// 7
// 9
```

The <span style="color: orange; font-weight: bold">break</span> keyword on the other hand stops the current iteration and moves out of the loop all together.

```go
for i := 0; i < 10; i++ {
  if i == 5 {
    break
  }
  fmt.Println(i)
}
// 0
// 1
// 2
// 3
// 4
```

##### Range

There's some syntactic sugar if you want to iterate over slices, strings or maps. You can omit index or element if you just one or the other.

```go
for INDEX, ELEMENT := range ITERABLE {
}

fruits := []string{"apple", "banana", "grape"}
for i, fruit := range fruits {
    fmt.Println(i, fruit)
}
// 0 apple
// 1 banana
// 2 grape
```