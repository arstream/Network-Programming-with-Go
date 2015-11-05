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
The address is usually written as four bytes in decimal with a dot `"."` between them, as in `127.0.0.1` or `66.102.11.104`.

The IP address of any device is generally composed of two parts: the address of the network in which the device resides, and the address of the device within that network. 
Once upon a time, the split between network address and internal address was simple and was based upon the bytes used in the IP address. 


* In a class A network, the first byte identifies the network, while the last three identify the device. There are only 128 class A networks, owned by the very early players in the internet space such as IBM, the General Electric Company and MIT [[1]](http://www.iana.org/assignments/ipv4-address-space/ipv4-address-space.xml)
    
* Class B networks use the first two bytes to identify the network and the last two to identify devices within the subnet. This allows upto $$2^{16}$$ (65,536) devices on a subnet
    
* Class C networks use the first three bytes to identify the network and the last one to identify devices within that network. This allows upto $$2^8$$ (actually 254, not 256) devices

This scheme doesn't work well if you want, say, 400 computers on a network. 254 is too small, while 65,536 is too large. 
In binary arithmetic terms, you want about 512. 
This can be achieved by using a 23 bit network address and 9 bits for the device addresses. 
Similarly, if you want upto 1024 devices, you use a 22 bit network address and a 10 bit device address.

Given an IP address of a device, and knowing how many bits N are used for the network address gives a relatively straightforward process for extracting the network address and the device address within that network. 
Form a "network mask" which is a 32-bit binary number with all ones in the first N places and all zeroes in the remaining ones. 
For example, if 16 bits are used for the network address, the mask is 11111111111111110000000000000000. 
It's a little inconvenient using binary, so decimal bytes are usually used. 
The netmask for 16 bit network addresses is 255.255.0.0, for 24 bit network addresses it is 255.255.255.0, while for 23 bit addresses it would be 255.255.254.0 and for 22 bit addresses it would be 255.255.252.0.

Then to find the network of a device, bit-wise AND it's IP address with the network mask, while the device address within the subnet is found with bit-wise AND of the 1's complement of the mask with the IP address. 


### IPv6 addresses

The internet has grown vastly beyond original expectations. The initially generous 32-bit addressing scheme is on the verge of running out. There are unpleasant workarounds such as NAT addressing, but eventually we will have to switch to a wider address space. IPv6 uses 128-bit addresses. Even bytes becomes cumbersome to express such addresses, so hexadecimal digits are used, grouped into 4 digits and separated by a colon `":"`. A typical address might be `2002:c0e8:82e7:0:0:0:c0e8:82e7`.

These addresses are not easy to remember! DNS will become even more important. There are tricks to reducing some addresses, such as eliding zeroes and repeated digits. For example, "localhost" is `0:0:0:0:0:0:0:1`, which can be shortened to `::1`.


