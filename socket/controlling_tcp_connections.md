# Controlling TCP connections

### Timeout

The server may wish to timeout a client if it does not respond quickly enough i.e. does not write a request to the server in time. This should be a long period (several minutes), because the user may be taking their time. Conversely, the client may want to timeout the server (after a much shorter time). 
Both do this by
```go
func (c *TCPConn) SetTimeout(nsec int64) os.Error
```

before any reads or writes on the socket.

### Staying alive

A client may wish to stay connected to a server even if it has nothing to send. It can use

```go
func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error
```

There are several other connection control methods, documented in the `"net"` package. 

