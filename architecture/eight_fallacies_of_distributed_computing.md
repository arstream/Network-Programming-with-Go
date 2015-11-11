## Eight fallacies of distributed computing

Sun Microsystems was a company that performed much of the early work in distributed systems, and even had a mantra "The network is the computer." Based on their experience over many years a number of the scientists at Sun came up with the following list of fallacies commonly assumed:

* The network is reliable.
* Latency is zero.
* Bandwidth is infinite.
* The network is secure.
* Topology doesn't change.
* There is one administrator.
* Transport cost is zero.
* The network is homogeneous.

Many of these directly impact on network programming. For example, the design of most remote procedure call systems is based on the premise that the network is reliable so that a remote procedure call will behave in the same way as a local call. The fallacies of zero latency and infinite bandwidth also lead to assumptions about the time duration of an RPC call being the same as a local call, whereas they are magnitudes of order slower.

The recognition of these fallacies led Java's RMI (remote method invocation) model to require every RPC call to potentially throw a ```RemoteException```. This forced programmers to at least recognise the possibility of network error and to remind them that they could not expect the same speeds as local calls. 