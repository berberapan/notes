#### If statement

If statements like Python are done without parentheses for the condition. But the block is enclosed by **{}**. Else and else if works as normal.

```go
if example > 5 {
  fmt.Println("Bigger than 5.")
}

if example > 5 {
  fmt.Println("Bigger than 5.")
} else if example > 0 {
  fmt.Println("Between 1 and 5")
} else {
  fmt.Println("Zero or negative")
}
```

You can also make a initial statement in an if or switch. Example below not the best but if you have a variable that get a return value true a function for example it can make the code a bit cleaner.

```go
if example := 6; example > 5 {
  fmt.Println("Bigger than 5.")
} 
```
***
#### Comparison operators

- **\==** equivalence operator
- **\!=** not equal
- **\<** less than
- **\>** greater than
- **\<\=** less than or equal
- **\>\=** greater than or equal

***
#### Logical operators

- **&&** is the AND operator
- **||** is the OR operator
- **!** is the NOT operator

***
#### Switch case

Syntax pretty similar to most other big languages switch cases. Default for any case that don't match and no need to break out after a case. 

```go
switch number {
  case 1:
    fmt.Println("I am number one!")
  case 2:
    fmt.Println("Two is not a winner")
  case 3:
    fmt.Println("Three no one remembers")
  default:
    fmt.Println("What does it take to be number one?")
}
```