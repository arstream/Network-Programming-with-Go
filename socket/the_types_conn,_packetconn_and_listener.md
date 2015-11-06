# The types Conn, PacketConn and Listener

 So far we have differentiated between the API for TCP and the API for UDP, using for example `DialTCP` and `DialUDP` returning a `TCPConn` and `UDPConn` respectively. The type `Conn` is an interface and both `TCPConn` and `UDPConn` implement this interface. To a large extent you can deal with this interface rather than the two types.

Instead of separate dial functions for TCP and UDP, you can use a single function

```go
func Dial(net, laddr, raddr string) (c Conn, err os.Error)
```
  
The net can be any of `"tcp"`, `"tcp4"` (IPv4-only), `"tcp6"` (IPv6-only), `"udp"`, `"udp4"` (IPv4-only), `"udp6"` (IPv6-only), `"ip"`, `"ip4"` (IPv4-only) and `"ip6"` IPv6-only). It will return an appropriate implementation of the `Conn` interface. Note that this function takes a string rather than address as `raddr` argument, so that programs using this can avoid working out the address type first.

Using this function makes minor changes to programs. For example, the earlier program to get *HEAD* information from a Web page can be re-written as

```go
/* IPGetHeadInfo
 */
package main

import (
	"bytes"
	"fmt"
	"io"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
		os.Exit(1)
	}
	service := os.Args[1]

	conn, err := net.Dial("tcp", service)
	checkError(err)

	_, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
	checkError(err)

	result, err := readFully(conn)
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

func readFully(conn net.Conn) ([]byte, error) {
	defer conn.Close()

	result := bytes.NewBuffer(nil)
	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		result.Write(buf[0:n])
		if err != nil {
			if err == io.EOF {
				break
			}
			return nil, err
		}
	}
	return result.Bytes(), nil
}
```

Writing a server can be similarly simplified using the function

```go 
func Listen(net, laddr string) (l Listener, err os.Error)
``` 
  
which returns an object implementing the Listener interface. This interface has a method

```go
func (l Listener) Accept() (c Conn, err os.Error)
```

which will allow a server to be built. Using this, the multi-threaded Echo server given earlier becomes

```go
/* ThreadedIPEchoServer
 */
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {

	service := ":1200"
	listener, err := net.Listen("tcp", service)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		go handleClient(conn)
	}
}

func handleClient(conn net.Conn) {
	defer conn.Close()

	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		if err != nil {
			return
		}
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
```

If you want to write a UDP server, then there is an interface `PacketConn` and a method to return an implementation of this:

```go
func ListenPacket(net, laddr string) (c PacketConn, err os.Error)
```

This interface has primary methods `ReadFrom` and `WriteTo` to handle packet reads and writes.

The Go net package recommends using these interface types rather than the concrete ones. But by using them, you lose specific methods such as `SetKeepAlive` or `TCPConn` and `SetReadBuffer` of `UDPConn`, unless you do a type cast. It is your choice. 


