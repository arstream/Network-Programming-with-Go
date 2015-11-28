# Chapter 5 Application-Level Protocols

A client and a server exchange messages consisting of message types and message data. This requires design of a suitable message exchange protocol. This chapter looks at some of the issues involved in this, and gives a complete example of a simple client-server application. 

## Introduction

A client and server need to exchange information via messages. TCP and UDP provide the transport mechanisms to do this. The two processes also need to have a protocol in place so that message exchange can take place meaningfully. A protocol defines what type of conversation can take place between two components of a distributed application, by specifying messages, data types, encoding formats and so on. 