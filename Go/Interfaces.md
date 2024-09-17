An interface type is defined as a set of method signatures.

Types implements interfaces implicit in Go. For a type to implement an interface it has to have got all the methods that are declared in the interface. This also means that every single type in Go also implements the empty interface `interface{}`.

Keep interfaces simple and avoid anti-patterns.

When you're creating an interface you create a type just like you do with a struct but use the keyword <span style="color: orange; font-weight: bold">interface</span> instead.

```go
type shape interface {
  area() float64
}

type rectangle struct {
  width, height float64
}

func (r rectangle) area() float64 {
  return r.width * r.height
}

type triangle struct {
	base, height float64
}

func (t triangle) area() float64 {
   return (t.base * t.height) / 2
}
```

When you have a function you can just use the shape type as a parameter if you're using example above. 

You're not required to name arguments for an interface but it can cause confusion if you don't.

```go
type Copier interface {
  Copy(string, string) int
}

// Compared to 

type Copier interface {
  Copy(sourceFile string, destinationFile string) (bytesCopied int)
}
```

##### Type assertion

Sometimes you want to access the underlying type of an interface value. You can then cast an interface to its underlying type with type assertion.

So for the example below we want to get the radius of a shape but we first need to assert that the shape is a circle for it to work. The value **ok** will be true if the value is a circle.

```go
type shape interface {
	area() float64
}

type circle struct {
	radius float64
}

c, ok := s.(circle)
if !ok {
	// log an error if s isn't a circle
	log.Fatal("s is not a circle")
}

radius := c.radius
```

##### Type switches

If you want to do a type assertion across several types you can use a type switch.
When you declare the type switch the syntax looks almost the same as for a type assertion but instead of having a specific type (like 'circle' in example above) you use the keyword <span style="color: orange; font-weight: bold">type</span>. Then in the cases you put in the specific types.

```go
func getShapeArea(i shape) float64 {
  switch value := i.(type) {
  case rectangle:
    return value.area()
  case triangle:
    return value.area()
  default:
    return 0.0
}
```