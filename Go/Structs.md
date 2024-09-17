Structs are used in Go to represent structured data. It's often useful to have different types of variables together. It can be compared with an object or a dictionary in other languages.

You use the keywords <span style="color: orange; font-weight: bold">type</span> and <span style="color: orange; font-weight: bold">struct</span> to define one. The variables and types are not separated by anything and just written out with a new line.

```go
type book struct {
  title string
  author string
  pages int
}
```

Structs can also be nested with other structs. They nested values can be accessed with dot operator. Like in the example below if I want the authors name I'd go `book.author.name`

```go
type book struct {
  title string
  author person
  pages int
}

type person stuct {
  name string
  age int
}
```

When designing a struct the order of types actually matters on memory usage. The easiest rule of thumb to not waste memory is to go biggest to smallest for the fields data types. You can also use the [reflect](https://pkg.go.dev/reflect) package to check the memory size.

##### Anonymous structs

You can use anonymous structs which means you don't define the name. This means you can't reference it again later in the code so it's only useful when you know something is only going to be used once. Named structs are usually easier to read so should be preferred but if you're sure it will only be used once and never again it can be useful to make an anonymous one.

```go
bookExample := struct {
  title string
  author string
  pages int
} {
  title: "Best book",
  author: "Joey Jo-Jo",
  pages: 1337,
}
```

As in the example above you need to instantiate the struct immediately with brackets after declaring it. Notice when you set the values you use a **,** to delimit and **:** to separate key and value.

##### Embedded structs

Similar to nested structs but accessed on a higher level are embedded structs.

```go
type author struct {
  name string
  age int
}

type book struct {
  title string
  author
  pages int
}
```

In the above example the author struct is embedded in the book struct. If I were to call the author's name I'd just have to write `book.name` instead of `book.author.name` like I would have to in a nested struct.

##### Struct methods

As Go isn't an object-oriented language it doesn't have traditional methods but methods can be tied to structs as a function that got a receiver. Receiver is the parameter that goes before the function name as shown below.

```go
type rectangle struct {
  width int
  height int
}

func (r rectangle) area() int {
  return r.width * r.height
}
```

