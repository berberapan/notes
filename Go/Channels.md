Go is a language that is designed to be concurrent. Spawning concurrent executions is quite straight forward, you just use the keyword **go**.

```go
go doSomething()
```

The doSomething will be executed concurrently with the rest of the code in the function. When using the go keyword you spawn a new **goroutine**.

##### Create a channel

Just like maps and slices, channels have to be created before use. Use the make keyword.

```go
ch := make(chan int)
```

##### Send and receive data

You use the **<-** operator. The data flows in the direction of the arrow. 

```go
ch <- 27 
```

The operation above will block until another goroutine is ready to receive the value. 

```go
v := <-ch
```

This would read and remove the value from the channel and save it to the v variable. This code will block until the channel got a value it can read.

When a group of goroutines are all blocking so none can continue it's called deadlock. This is a common bug to watch out for.

##### Buffered channels

You can set a buffer to your channels. The length will be the int you set when you make the channel.

```go
ch := make(chan int, 100)
```

A channel with a buffer won't block until the buffer is full.