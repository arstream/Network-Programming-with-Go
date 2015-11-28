## Handling Timeouts

Because these channels use the network, there is alwasy the possibility of network errors leading to [timeouts](http://blog.golang.org/2010/09/go-concurrency-patterns-timing-out-and.html). 
Andrew Gerrand points out a solution using timeouts: "[Set up a timeout channel.] We can then use a select statement to receive from either ch or timeout. If nothing arrives on ch after one second, the timeout case is selected and the attempt to read from ch is abandoned."

```go
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1e9) // one second
    timeout <- true
}()

select {
case <- ch:
    // a read from ch has occurred
case <- timeout:
    // the read from ch has timed out
}
```



