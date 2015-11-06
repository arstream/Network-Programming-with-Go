# TCP Sockets

When you know how to reach a service via its network and port IDs, what then? 
If you are a client you need an API that will allow you to connect to a service and then to send messages to that service and read replies back from the service.

If you are a server, you need to be able to bind to a port and listen at it. When a message comes in you need to be able to read it and write back to the client.

The `net.TCPConn` is the Go type which allows full duplex communication between the client and the server. Two major methods of interest are

func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)   
    

A TCPConn is used by both a client and a server to read and write messages. 