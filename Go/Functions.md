To create a function you use the <span style="color: orange; font-weight: bold">func</span> keyword. Just like the variables you set the data type last.

```go
func add(x int, y int) int {
  return x + y
}
```

##### Multiple parameters

When you have multiple parameters of the same type you only need to declare the type once if you put them in order.

```go
func example(num1, num2 int, text string)
```

##### Variables passed by value

In Go most data types are passed by value. When a variable is passed into a function, the function receives a copy of the value and the function won't mutate the original data.

```go
func main() {
  num := 1
  addTo(num)

  fmt.Println(num)
  // This will print 1. The addTo() recieved a copy of num and is not mutating
  // the num value.

}

func addTo(num int) {
  num++
}
```

##### Ignore return values

If a function returns a value you're not interested in using you can ignore the return. This comes in handy with functions with multiple returns. You use the **\_** to ignore a value.

```go
func example() (x,y int) {
  return 1, 2
}

x, _ := example()

//This will only return x.
```

##### Named return values

When you're writing a function you can give the return values names. It will work in the same way as declaring variable names inside the function. The return statement can then just be on its own and it will return the named values.

It's called naked return statement if you don't type out the return values and the Go documentation suggests they should only be used in short functions as the readability for a longer function would be poor.

If you have more than one return it's recommended to use named return values as it makes for better readability.

```go
func example() (num1, num2 int) {
  return
} 

// This returns num1 and num2. In this example both would be initalised as 0.
```

##### Early returns

Go supports returns early in a function. Example below shows how an error can be handled for divide with zero by returning the error value early.

```go
func divide(dividend, divisor int) (int, error) {
  if divisor == 0 {
    return 0, errors.New("Can't divide by 0")
  }
  return dividend/divisor, nil
}
```

It can also make code more readable when you don't need to have nested if/else but can just return a value in certain conditions.