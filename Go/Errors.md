There's a built in error type in Go, it's a type that implements the error interface. If something goes wrong the function should return an error as its last return value. If calling on a function that can return an error the best practice is to test if the error is returning **nil**.

```go
intValue, err = strconv.Atoi("9")
if err != nil {
  log.Println("Unable to convert value")
  return err
}
```

A nil value on the err means that no errors occurred while any other value would mean a failure. 

##### Make own errors

As errors are tied to the error interface you can make your own custom errors. The error interface looks like below. So your own error needs to have a method with the name of Error() that returns a string.

```go 
type error interface {
    Error() string
}
```

So if making an error for dividing with zero it can look like this

```go
type divideZeroError struct {
   dividend float64
}

func (e dividZeroError) Error() string {
  return fmt.Sprintf("%v can not be divided by 0", e.dividend)
}

func divide(dividend, divisor float64) (float64, error) {
	if divisor == 0 {
		return 0.0, divideZeroError{dividend: dividend}
	}
	return dividend / divisor, nil
}
```

There's also a standard library in Go that can be used that's a bit easier to use.
`import "errors"` and then you can just use the New() function. If making the above example with this library instead it would look like this.

```go
func divide(dividend, divisor float64) (float64, error) {
  if divisor == 0 {
    return 0.0, errors.New("Value can't be divided by 0")
  }
  return dividend / divisor, nil
}
```
