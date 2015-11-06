# Server listening on multiple sockets

A server may be attempting to listen to multiple clients not just on one port, but on many. In this case it has to use some sort of polling mechanism between the ports.

In C, the `select()` call lets the kernel do this work. The call takes a number of file descriptors. The process is suspended. When I/O is ready on one of these, a wakeup is done, and the process can continue. This is cheaper than busy polling. In Go, accomplish the same by using a different goroutine for each port. A thread will become runnable when the lower-level `select()` discovers that I/O is ready for this thread. 