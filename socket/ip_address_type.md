# IP address type

The package `"net"` defines many types, functions and methods of use in Go network programming. The type `IP` is defined as an array of bytes 
```go
type IP []byte
```
There are several functions to manipulate a variable of type IP, but you are likely to use only some of them in practice. 
For example, the function ParseIP(String) will take a dotted IPv4 address or a colon IPv6 address, while the IP method String will return a string. 
Note that you may not get back what you started with: the string form of `0:0:0:0:0:0:0:1` is `::1`.

A program to illustrate this is 

```go
/* IP
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s ip-addr\n", os.Args[0])
		os.Exit(1)
	}
	name := os.Args[1]

	addr := net.ParseIP(name)
	if addr == nil {
		fmt.Println("Invalid address")
	} else {
		fmt.Println("The address is ", addr.String())
	}
	os.Exit(0)
}
```

If this is compiled to the executable `IP` then it can run for example as 

    IP 127.0.0.1
    
with response

    The address is 127.0.0.1
    
or as

    IP 0:0:0:0:0:0:0:1
    
with response

    The address is ::1
    
### The type IPmask

In order to handle masking operations, there is the type

```go
type IPMask []byte
```

There is a function to create a mask from a 4-byte IPv4 address

```go
func IPv4Mask(a, b, c, d byte) IPMask
```

Alternatively, there is a method of `IP` which returns the default mask

```go    
func (ip IP) DefaultMask() IPMask
```

Note that the string form of a mask is a hex number such as `ffff0000` for a mask of `255.255.0.0`.

A mask can then be used by a method of an IP address to find the network for that IP address

```go
func (ip IP) Mask(mask IPMask) IP
```
    
An example of the use of this is the following program: 
    
```go
/* Mask
 */

package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s dotted-ip-addr\n", os.Args[0])
		os.Exit(1)
	}
	dotAddr := os.Args[1]

	addr := net.ParseIP(dotAddr)
	if addr == nil {
		fmt.Println("Invalid address")
		os.Exit(1)
	}
	mask := addr.DefaultMask()
	network := addr.Mask(mask)
	ones, bits := mask.Size()
	fmt.Println("Address is ", addr.String(),
		" Default mask length is ", bits,
		"Leading ones count is ", ones,
		"Mask is (hex) ", mask.String(),
		" Network is ", network.String())
	os.Exit(0)
}
```

If this is compiled to `Mask` and run by

    Mask 127.0.0.1
    
it will return

    Address is  127.0.0.1  Default mask length is  8  Network is  127.0.0.0
    

### The type IPAddr

Many of the other functions and methods in the net package return a pointer to an `IPAddr`. 
This is simply a structure containing an IP.

```go    
type IPAddr {
    IP IP
}
```
  
A primary use of this type is to perform DNS lookups on IP host names.

```go
func ResolveIPAddr(net, addr string) (*IPAddr, os.Error)
```
    
where `net` is one of `"ip"`, `"ip4"` or `"ip6"`. This is shown in the program 

```go

/* ResolveIP
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s hostname\n", os.Args[0])
		fmt.Println("Usage: ", os.Args[0], "hostname")
		os.Exit(1)
	}
	name := os.Args[1]

	addr, err := net.ResolveIPAddr("ip", name)
	if err != nil {
		fmt.Println("Resolution error", err.Error())
		os.Exit(1)
	}
	fmt.Println("Resolved address is ", addr.String())
	os.Exit(0)
}
```

Running `ResolveIP www.google.com` returns

    Resolved address is  66.102.11.104
    

### Host lookup

The function `ResolveIPAddr` will perform a DNS lookup on a hostname, and return a single IP address. 
However, hosts may have multiple IP addresses, usually from multiple network interface cards. They may also have multiple host names, acting as aliases.

```go
func LookupHost(name string) (cname string, addrs []string, err os.Error)
```
    
One of these addresses will be labeled as the "canonical" host name. If you wish to find the canonical name, use 

```go
func LookupCNAME(name string) (cname string, err os.Error)
```

This is shown in the following program     

```go
/* LookupHost
 */

package main

import (
	"net"
	"os"
	"fmt"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s hostname\n", os.Args[0])
		os.Exit(1)
	}
	name := os.Args[1]

	addrs, err := net.LookupHost(name)
	if err != nil {
		fmt.Println("Error: ", err.Error())
		os.Exit(2)
	}

	for _, s := range addrs {
		fmt.Println(s)
	}
	os.Exit(0)
}
```
    
Note that this function returns strings, not `IPAddress` values.
    


























    



    

