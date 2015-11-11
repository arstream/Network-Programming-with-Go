## Server Distribution

A client-server systems need not be simple. The basic model is single client, single server 

![one-one](../assets/one-one.gif)

but you can also have multiple clients, single server 

![many-one](../assets/many-one.gif)

In this, the master receives requests and instead of handling them one at a time itself, passes them off to other servers to handle. This is a common model when concurrent clients are possible.

There are also single client, multiple servers 

![one-many](../assets/one-many.gif)

which occurs frequently when a server needs to act as a client to other servers, such as a business logic server getting information from a database server. And of course, there could be multiple clients with multiple servers.
