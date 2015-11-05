# IP address type

The package `"net"` defines many types, functions and methods of use in Go network programming. The type `IP` is defined as an array of bytes 

    type IP []byte
    
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
    
  

Note that the string form of a mask is a hex number such as ffff0000 for a mask of 255.255.0.0.

A mask can then be used by a method of an IP address to find the network for that IP address

    
func (ip IP) Mask(mask IPMask) IP
    
  

An example of the use of this is the following program: 
    


    

