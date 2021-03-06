# Manners

A *polite* webserver for Go.

Manners allows you to shut your Go webserver down gracefully, without dropping any requests. It can act as a drop-in replacement for the standard library's http.ListenAndServe function:

```go
func main() {
  handler = MyHTTPHandler()
  signal.Notify(manners.ShutdownChannel)
  manners.ListenAndServe(handler, ":7000")
}
```

Advanced users have full access to Manners' internals, so they can construct custom handling procedures:

```go
func main() {
  handler = MyHTTPHandler()
  baseListener, err := net.Listen(":7000")
  if err != nil {
    panic(err)
  }
  listener := manners.NewListener(baseListener)

  // Do all sorts of stuff with the listener

  manners.Serve(listener, handler)
}
```

It's also easy to trigger the shutdown command programmically:

```go
manners.ShutdownChannel <- syscall.SIGINT
```

Manners ensures that all requests are served by incrementing a waitgroup when a request comes in and decrementing it when the request finishes.

If your request handler spawns Goroutines that are not guaranteed to finish with the request, you can ensure they are also completed with the `StarRoutine` function:

```go
func (this *MyHTTPHandler) ServeHTTP(response http.ResponseWriter, request *http.Request) {
  DoAsynchronousComputations()
  // Implicitly return 200
}

func DoAsynchronousComputations() {
  manners.StartRoutine()
  go func() {
    defer manners.FinishRoutine()
    // Do the computations
  }()
}
```

### Installation

`go get github.com/braintree/manners`

### Contributors

- [Lionel Barrow](http://github.com/lionelbarrow)
- [Paul Rosenzweig](http://github.com/paulrosenzweig)
