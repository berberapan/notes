The format package for Go. This package is for formatting input and output.
#### Println()

Prints out to the standard output. If more than one variable is added it will be a space added between them.

```go
text := "Hello"
text2 := "World"
fmt.Println(text, text2)

 >> Hello World
```

***
#### Printf()

Prints out a formatted line to standard output. Read more about formatting options in Sprintf(). Does not add a newline at the end like Println().

```go
name := "Joe"
fmt.Printf("Hello %v", name)

 >> Hello Joe
```

***
#### Sprintf()

Like the print version but returns the string instead of printing. 

- %v - Default format (easiest to use)
- %t - Boolean
- %d - Normal integer
- %f - Decimal point (%.2f for two decimals)
- %s - String

[All formatting options](https://pkg.go.dev/fmt#hdr-Printing)

```go
number := 0.12345
fmt.Sprintf("%.2f", number)

 >> "0.12"
```

***




