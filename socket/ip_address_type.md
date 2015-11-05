# IP address type

The package `"net"` defines many types, functions and methods of use in Go network programming. The type `IP` is defined as an array of bytes 

    type IP []byte
    
There are several functions to manipulate a variable of type IP, but you are likely to use only some of them in practice. 
For example, the function ParseIP(String) will take a dotted IPv4 address or a colon IPv6 address, while the IP method String will return a string. 
Note that you may not get back what you started with: the string form of `0:0:0:0:0:0:0:1` is `::1`.

A program to illustrate this is 