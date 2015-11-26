# Chapter 13 Remote Procedure Call

## Introduction

 Socket and HTTP programming use a message-passing paradigm. A client sends a message to a server which usually sends a message back. Both sides ae responsible for creating messages in a format understood by both sides, and in reading the data out of those messages.

However, most standalone applications do not make so much use of message passing techniques. Generally the preferred mechanism is that of the function (or method or procedure) call. In this style, a program will call a function with a list of parameters, and on completion of the function call will have a set of return values. These values may be the function value, or if addresses have been passed as parameters then the contents of those addresses might have been changed.

The remote procedure call is an attempt to bring this style of programming into the network world. Thus a client will make what looks to it like a normal procedure call. The client-side will package this into a network message and transfer it to the server. The server will unpack this and turn it back into a procedure call on the server side. The results of this call will be packaged up for return to the client.

Diagrammatically it looks like 



