# Chapter 13 Remote Procedure Call

## Introduction

 Socket and HTTP programming use a message-passing paradigm. A client sends a message to a server which usually sends a message back. Both sides ae responsible for creating messages in a format understood by both sides, and in reading the data out of those messages.

However, most standalone applications do not make so much use of message passing techniques. Generally the preferred mechanism is that of the function (or method or procedure) call. In this style, a program will call a function with a list of parameters, and on completion of the function call will have a set of return values. These values may be the function value, or if addresses have been passed as parameters then the contents of those addresses might have been changed.

The remote procedure call is an attempt to bring this style of programming into the network world. Thus a client will make what looks to it like a normal procedure call. The client-side will package this into a network message and transfer it to the server. The server will unpack this and turn it back into a procedure call on the server side. The results of this call will be packaged up for return to the client.

Diagrammatically it looks like 

![rpc_stub](../assets/rpc_stub.png)

where the steps are

1. The client calls the local stub procedure. The stub packages up the parameters into a network message. This is called marshalling.
2. Networking functions in the O/S kernel are called by the stub to send the message.
3. The kernel sends the message(s) to the remote system. This may be connection-oriented or connectionless.
4. A server stub unmarshalls the arguments from the network message.
5. The server stub executes a local procedure call.
6. The procedure completes, returning execution to the server stub.
7. The server stub marshals the return values into a network message.
8. The return messages are sent back.
9. The client stub reads the messages using the network functions.
10. The message is unmarshalled. and the return values are set on the stack for the local process.

There are two common styles for implementing RPC. The first is typified by Sun's RPC/ONC and by CORBA. In this, a specification of the service is given in some abstract language such as CORBA IDL (interface definition language). This is then compiled into code for the client and for the server. The client then writes a normal program containing calls to a procedure/function/method which is linked to the generated client-side code. The server-side code is actually a server itself, which is linked to the procedure implementation that you write.

In this way, the client-side code is almost identical in appearance to a normal procedure call. Generally there is a little extra code to locate the server. In Sun's ONC, the address of the server must be known; in CORBA a naming service is called to find the address of the server; In Java RMI, the IDL is Java itself and a naming service is used to find the address of the service.

In the second style, you have to make use of a special client API. You hand the function name and its parameters to this library on the client side. On the server side, you have to explicitly write the server yourself, as well as the remote procedure implementation.

This approach is used by many RPC systems, such as Web Services. It is also the approach used by Go's RPC. 



