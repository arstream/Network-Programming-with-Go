# Services

Services run on host machines. They are typically long lived and are designed to wait for requests and respond to them. There are many types of services, and there are many ways in which they can offer their services to clients. The internet world bases many of these services on two methods of communication, TCP and UDP, although there are other communication protocols such as SCTP waiting in the wings to take over. Many other types of service, such as peer-to-peer, remote procedure calls, communicating agents, and many others are built on top of TCP and UDP.
Ports

Services live on host machines. The IP address will locate the host. But on each computer may be many services, and a simple way is needed to distinguish between them. The method used by TCP, UDP, SCTP and others is to use a port number. This is an unsigned integer beween 1 and 65,535 and each service will associate itself with one or more of these port numbers.

There are many "standard" ports. Telnet usually uses port 23 with the TCP protocol. DNS uses port 53, either with TCP or with UDP. FTP uses ports 21 and 20, one for commands, the other for data transfer. HTTP usually uses port 80, but it often uses ports 8000, 8080 and 8088, all with TCP. The X Window System often takes ports 6000-6007, both on TCP and UDP.

On a Unix system, the commonly used ports are listed in the file `/etc/services`. Go has a function to interrogate this file 

```go
func LookupPort(network, service string) (port int, err os.Error)
```

The network argument is a string such as `"tcp"` or `"udp"`, while the service is a string such as `"telnet"` or `"domain"` (for DNS).

A program using this is 

```go
/* LookupPort
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 3 {
		fmt.Fprintf(os.Stderr,
			"Usage: %s network-type service\n",
			os.Args[0])
		os.Exit(1)
	}
	networkType := os.Args[1]
	service := os.Args[2]

	port, err := net.LookupPort(networkType, service)
	if err != nil {
		fmt.Println("Error: ", err.Error())
		os.Exit(2)
	}

	fmt.Println("Service port ", port)
	os.Exit(0)
}
```

For example, running `LookupPort tcp telnet` prints `Service port: 23`.


### The type TCPAddr

The type `TCPAddr` is a structure containing an IP and a port:

```go
type TCPAddr struct {
    IP   IP
    Port int
}
```
    
The function to create a `TCPAddr` is `ResolveTCPAddr`

```go
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```
  
where `net` is one of `"tcp"`, `"tcp4"` or `"tcp6"` and the `addr` is a string composed of a host name or IP address, followed by the port number after a `":"`, such as `"www.google.com:80"` or `"127.0.0.1:22"`. 
If the address is an IPv6 address, which already has colons in it, then the host part must be enclosed in square brackets, such as `"[::1]:23"`. 
Another special case is often used for servers, where the host address is zero, so that the TCP address is really just the port name, as in `":80"` for an HTTP server. 