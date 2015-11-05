# Internet addresses

In order to use a service you must be able to find it. 
The Internet uses an address scheme for devices such as computers so that they can be located. 
This addressing scheme was originally devised when there were only a handful of connected computers, and very generously allowed upto $$2^{32}$$ addresses, using a 32 bit unsigned integer. 
These are the so-called IPv4 addresses. 
 
In recent years, the number of connected (or at least directly addressable) devices has threatened to exceed this number, and so "any day now" we will switch to IPv6 addressing which will allow upto $$2^{128}$$ addresses, using an unsigned 128 bit integer. 
The changeover is most likely to be forced by emerging countries, as the developed world has already taken nearly all of the pool of IPv4 addresses.

### IPv4 addresses

The address is a 32 bit integer which gives the IP address. 
This addresses down to a network interface card on a single device. 
The address is usually written as four bytes in decimal with a dot `"."` between them, as in `"127.0.0.1"` or `"66.102.11.104"`.

The IP address of any device is generally composed of two parts: the address of the network in which the device resides, and the address of the device within that network. 
Once upon a time, the split between network address and internal address was simple and was based upon the bytes used in the IP address. 