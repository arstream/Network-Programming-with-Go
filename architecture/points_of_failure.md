## Points of Failure

Distributed applications run in a complex environment. This makes them much more prone to failure than standalone applications on a single computer. The points of failure include

* The client side of the application could crash
* The client system may have h/w problems
* The client's network card could fail
* Network contention could cause timeouts
* There may be network address conflicts
* Network elements such as routers could fail
* Transmission errors may lose messages
* The client and server versions may be incompatible
* The server's network card could fail
* The server system may have h/w problems
* The server s/w may crash
* The server's database may become corrupted 

Applications have to be designed with these possible failures in mind. Any action performed by one component must be recoverable if failure occurs in some other part of the system. Techniques such as transactions and continuous error checking need to be employed to avoid errors. 