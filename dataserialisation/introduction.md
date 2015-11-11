## Introduction

A client and server need to exchange information via messages. TCP and UDP provide the transport mechanisms to do this. The two processes also have to have a protocol in place so that message exchange can take place meaningfully.

Messages are sent across the network as a sequence of bytes, which has no structure except for a linear stream of bytes. We shall address the various possibilities for messages and the protocols that define them in the next chapter. In this chapter we concentrate on a component of messages - the data that is transferred.

A program will typically build complex data structures to hold the current program state. In conversing with a remote client or service, the program will be attempting to transfer such data structures across the network - that is, outside of the application's own address space.

Programming languages use structured data such as

* records/structures
* variant records
* array - fixed size or varying
* string - fixed size or varying
* tables - e.g. arrays of records
* non-linear structures such as
    * circular linked list
    * binary tree
    * objects with references to other objects


None of IP, TCP or UDP packets know the meaning of any of these data types. All that they can contain is a sequence of bytes. 
Thus an application has to *serialise* any data into a stream of bytes in order to write it, and *deserialise* the stream of bytes back into suitable data structures on reading it. These two operations are known as *marshalling* and *unmarshalling* respectively.

For example, consider sending the following variable length table of two columns of variable length strings: 

|  |  |
| -- | -- |
| fred | programmer |
| liping | analyst |
| sureerat | manager |


This could be done by in various ways. For example, suppose that it is known that the data will be an unknown number of rows in a two-column table. Then a marshalled form could be

    3                // 3 rows, 2 columns assumed
    4 fred           // 4 char string,col 1
    10 programmer    // 10 char string,col 2
    6 liping         // 6 char string, col 1
    7 analyst        // 7 char string, col 2
    8 sureerat       // 8 char string, col 1
    7 manager        // 7 char string, col 2

Variable length things can alternatively have their length indicated by terminating them with an "illegal" value, such as `'\0'` for strings:

    3
    fred\0        
    programmer\0
    liping\0
    analyst\0
    sureerat\0
    manager\0

Alternatively, it may be known that the data is a 3-row fixed table of two columns of strings of length 8 and 10 respectively. Then a serialisation could be

    fred\0\0\0\0
    programmer
    liping\0\0
    analyst\0\0\0
    sureerat
    manager\0\0\0

 Any of these formats is okay - but the message exchange protocol must specify which one is used, or allow it to be determined at runtime.