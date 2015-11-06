# TCP Sockets

When you know how to reach a service via its network and port IDs, what then? 
If you are a client you need an API that will allow you to connect to a service and then to send messages to that service and read replies back from the service.

If you are a server, you need to be able to bind to a port and listen at it. When a message comes in you need to be able to read it and write back to the client.

The `net.TCPConn` is the Go type which allows full duplex communication between the client and the server. Two major methods of interest are

```go
func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)   
```
    
A `TCPConn` is used by both a client and a server to read and write messages. 

### TCP client

Once a client has established a TCP address for a service, it "dials" the service. If successful, the dial returns a `TCPConn` for communication. The client and the server exchange messages on this. Typically a client writes a request to the server using the `TCPConn`, and reads a response from the `TCPConn`. This continues until either (or both) sides close the connection. A TCP connection is established by the client using the function

```go
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)
```

where `laddr` is the local address which is usually set to `nil` and `raddr` is the remote address of the service, and the `net` string is one of `"tcp4"`, `"tcp6"` or `"tcp"` depending on whether you want a TCPv4 connection, a TCPv6 connection or don't care.

A simple example can be provided by a client to a web (HTTP) server. We will deal in substantially more detail with HTTP clients and servers in a later chapter, but for now we will keep it simple.

One of the possible messages that a client can send is the `"HEAD"` message. This queries a server for information about the server and a document on that server. The server returns information, but does not return the document itself. The request sent to query an HTTP server could be

```"HEAD / HTTP/1.0\r\n\r\n"```
 
which asks for information about the root document and the server. A typical response might be

```
HTTP/1.0 200 OK
ETag: "-9985996"
Last-Modified: Thu, 25 Mar 2010 17:51:10 GMT
Content-Length: 18074
Connection: close
Date: Sat, 28 Aug 2010 00:43:48 GMT
Server: lighttpd/1.4.23
``` 

We first give the program (`GetHeadInfo.go`) to establish the connection for a TCP address, send the request string, read and print the response. Once compiled it can be invoked by e.g.

```GetHeadInfo www.google.com:80```

The program is

```go
/* GetHeadInfo
 */
package main

import (
	"net"
	"os"
	"fmt"
	"io/ioutil"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s host:port ", os.Args[0])
		os.Exit(1)
	}
	service := os.Args[1]

	tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
	checkError(err)

	conn, err := net.DialTCP("tcp", nil, tcpAddr)
	checkError(err)

	_, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
	checkError(err)

	//result, err := readFully(conn)
	result, err := ioutil.ReadAll(conn)
	checkError(err)

	fmt.Println(string(result))

	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
```

The first point to note is the almost excessive amount of error checking that is going on. This is normal for networking programs: the opportunities for failure are substantially greater than for standalone programs. Hardware may fail on the client, the server, or on any of the routers and switches in the middle; communication may be blocked by a firewall; timeouts may occur due to network load; the server may crash while the client is talking to it. The following checks are performed:

* There may be syntax errors in the address specified
* The attempt to connect to the remote service may fail. For example, the service requested might not be running, or there may be no such host connected to the network
* Although a connection has been established, writes to the service might fail if the connection has died suddenly, or the network times out
* Similarly, the reads might fail

Reading from the server requires a comment. In this case, we read essentially a single response from the server. This will be terminated by end-of-file on the connection. However, it may consist of several TCP packets, so we need to keep reading till the end of file. The `io/ioutil` function `ReadAll` will look after these issues and return the complete response. (Thanks to Roger Peppe on the golang-nuts mailing list.).

There are some language issues involved. First, most of the functions return a dual value, with possible error as second value. If no error occurs, then this will be nil. In C, the same behaviour is gained by special values such as `NULL`, or `-1`, or zero being returned - if that is possible. In Java, the same error checking is managed by throwing and catching exceptions, which can make the code look very messy.

In earlier versions of this program, I returned the result in the array `buf`, which is of type `[512]byte`. Attempts to coerce this to a string failed - only byte arrays of type `[]byte` can be coerced. This is a bit of a nuisance. 


### A Daytime server

About the simplest service that we can build is the daytime service. This is a standard Internet service, defined by *RFC 867*, with a default port of 13, on both TCP and UDP. Unfortunately, with the (justified) increase in paranoia over security, hardly any sites run a daytime server any more. Never mind, we can build our own. (For those interested, if you install *inetd* on your system, you usually get a daytime server thrown in.)

A server registers itself on a port, and listens on that port. Then it blocks on an "accept" operation, waiting for clients to connect. When a client connects, the accept call returns, with a connection object. The daytime service is very simple and just writes the current time to the client, closes the connection, and resumes waiting for the next client.

The relevant calls are

```go
func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)
func (l *TCPListener) Accept() (c Conn, err os.Error)
```

The argument `net` can be set to one of the strings `"tcp"`, `"tcp4"` or `"tcp6"`. The IP address should be set to zero if you want to listen on all network interfaces, or to the IP address of a single network interface if you only want to listen on that interface. If the port is set to zero, then the O/S will choose a port for you. Otherwise you can choose your own. Note that on a Unix system, you cannot listen on a port below 1024 unless you are the system supervisor, root, and ports below 128 are standardized by the *IETF*. The example program chooses port 1200 for no particular reason. The TCP address is given as `":1200"` - all interfaces, port 1200.

The program is

```go
/* DaytimeServer
 */
package main

import (
	"fmt"
	"net"
	"os"
	"time"
)

func main() {

	service := ":1200"
	tcpAddr, err := net.ResolveTCPAddr("ip4", service)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}

		daytime := time.Now().String()
		conn.Write([]byte(daytime)) // don't care about return value
		conn.Close()                // we're finished with this client
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
```

If you run this server, it will just wait there, not doing much. When a client connects to it, it will respond by sending the daytime string to it and then return to waiting for the next client.

Note the changed error handling in the server as compared to a client. The server should run forever, so that if any error occurs with a client, the server just ignores that client and carries on. A client could otherwise try to mess up the connection with the server, and bring it down!

We haven't built a client. That is easy, just changing the previous client to omit the initial write. Alternatively, just open up a telnet connection to that host:

```telnet localhost 1200```
    
This will produce output such as

```
$telnet localhost 1200
Trying ::1...
Connected to localhost.
Escape character is '^]'.
Sun Aug 29 17:25:19 EST 2010Connection closed by foreign host.
```    

where `"Sun Aug 29 17:25:19 EST 2010"` is the output from the server. 


### Multi-threaded server

"echo" is another simple IETF service. This just reads what the client types, and sends it back:


/* SimpleEchoServer
 */
package main

import (
	"net"
	"os"
	"fmt"
)

func main() {

	service := ":1201"
	tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		handleClient(conn)
		conn.Close() // we're finished
	}
}

func handleClient(conn net.Conn) {
	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		if err != nil {
			return
		}
		fmt.Println(string(buf[0:]))
		_, err2 := conn.Write(buf[0:n])
		if err2 != nil {
			return
		}
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}

While it works, there is a significant issue with this server: it is single-threaded. While a client has a connection open to it, no other cllient can connect. Other clients are blocked, and will probably time out. Fortunately this is easly fixed by making the client handler a go-routine. We have also moved the connection close into the handler, as it now belongs there


/* ThreadedEchoServer
 */
package main

import (
	"net"
	"os"
	"fmt"
)

func main() {

	service := ":1201"
	tcpAddr, err := net.ResolveTCPAddr("ip4", service)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		// run as a goroutine
		go handleClient(conn)
	}
}

func handleClient(conn net.Conn) {
	// close connection on exit
	defer conn.Close()

	var buf [512]byte
	for {
		// read upto 512 bytes
		n, err := conn.Read(buf[0:])
		if err != nil {
			return
		}

		// write the n bytes read
		_, err2 := conn.Write(buf[0:n])
		if err2 != nil {
			return
		}
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
