#### Types

These are the basic variable types

```go 
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128

```

The different bits to the number variables should just be used if needed. If you just want to use a normal int you don't need to set the bits.

When working with individual characters you should use **runes**. As stated above they are an alias for int32 so can hold any type of character, e.g. emojis and Asian signs.

#### Declaring Variable

You use the keyword <span style="color: orange; font-weight: bold">var</span> to declare a variable. Declaration of the type happens after the variable name.

```go
var number int = 10
```

There's also a short way do declare a variable if you're inside a function. You only need the variable name followed by **:=**. The type will be inferred by the value you set.

```go
number := 10
```

You can also declare multiple variable on the same line.

```go 
number, name := 10, "Joe"
```

If you want to set a constant you need to use the keyword <span style="color: orange; font-weight: bold">const</span>. Constants can't use the := to set the variable and they can obviously not be changed after set. Can be set as below outside and in functions. Only character, string, boolean and numeric values can be constants. They also need to be known at compile time.

```go
const myFaveNum = 9
```