## Connection Models

In order for two computers to communicate, they must set up a path whereby they can send at least one message in a session. There are two major models for this:

* Connection oriented
* Connectionless

### Connection oriented

A single connection is established for the session. Two-way communications flow along the connection. When the session is over, the connection is broken. The analogy is to a phone conversation. An example is TCP

### Connectionless

In a connectionless system, messages are sent independant of each other. Ordinary mail is the analogy. Connectionless messages may arrive out of order. An example is the IP protocol. 

Connection oriented transports may be established on top of connectionless ones - TCP over IP. Connectionless transports my be established on top of connection oriented ones - HTTP over TCP.

There can be variations on these. For example, a session might enforce messages arriving, but might not guarantee that they arrive in the order sent. However, these two are the most common. 