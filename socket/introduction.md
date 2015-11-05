# Introduction

There are many kinds of networks in the world. 
These range from the very old such as serial links, through to wide area networks made from copper and fiber, to wireless networks of various kinds, both for computers and for telecommunications devices such as phones. 
These networks obviously differ at the physical link layer, but in many cases they also differed at higher layers of the OSI stack.

Over the years there has been a convergence to the "internet stack" of IP and TCP/UDP. 
For example, Bluetooth defines physical layers and protocol layers, but on top of that is an IP stack so that the same internet programming techniques can be employed on many Bluetooth devices. 
Similarly, developing 4G wireless phone technologies such as LTE (Long Term Evolution) will also use an IP stack.

While IP provides the networking layer 3 of the OSI stack, TCP and UDP deal with layer 4. 
These are not the final word, even in the internet world: SCTP has come from the telecommunications to challenge both TCP and UDP, while to provide internet services in interplanetary space requires new, under development protocols such as DTN. 
Nevertheless, IP, TCP and UDP hold sway as principal networking technologies now and at least for a considerable time into the future. Go has full support for this style of programming

This chapter shows how to do TCP and UDP programming using Go, and how to use a raw socket for other protocols. 