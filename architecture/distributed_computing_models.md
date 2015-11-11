## Distributed Computing Models

At the highest level, we could consider the equivalence or the non-equivalence of components of a distributed system. The most common occurrence is an asymmetric one: a client sends requests to a server, and the server responds. This is a *client-server* system.

If both components are equivalent, both able to initiate and to respond to messages, then we have a *peer-to-peer* system. Note that this is a logical classification: one peer may be a 16,000 core mainframe, the other might be a mobile phone. But if both can act similarly then they are peers.

A third model is the so-called *filter*. Here one component passes information to another which modifies it before passing it to a third. This is a fairly common model: for example, the middle component gets information from a database as SQL records and transforms it into an HTML table for the third component (which might be a browser).

These are illustrated as: 

![peer](../assets/peer.gif)