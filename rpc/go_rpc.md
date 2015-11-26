## Go RPC

Go's RPC is so far unique to Go. It is different to the other RPC systems, so a Go client will only talk to a Go server. It uses the Gob serialisation system discussed in chapter X, which defines the data types which can be used.

RPC systems generally make some restrictions on the functions that can be called across the network. This is so that the RPC system can properly determine what are value arguments to be sent, what are reference arguments to receive answers, and how to signal errors.

In Go, the restriction is that

* the function must be public (begin with a capital letter);
* have exactly two arguments, the first is a pointer to value data to be received by the function from the client, and the second is a pointer to hold the answers to be returned to the client; and
* have a return value of type `error`

For example, a valid function is

```
F(&T1, &T2) error
``` 

The restriction on arguments means that you typically have to define a structure type. Go's RPC uses the `gob` package for marshalling and unmarshalling data, so the argument types have to follow the rules of `gob` as discussed in an earlier chapter.

We shall follow the example given in the Go documentation, as this illustrates the important points. The server performs two operations which are trivial - they do not require the "grunt" of RPC, but are simple to understand. The two operations are to multiply two integers, and the second is to find the quotient and remainder after dividing the first by the second.

The two values to be manipulated are given in a structure:

```go
type Values struct {
    X, Y int
}
```
    
The sum is just an `int`, while the quotient/remainder is another structure

```go
type Quotient struct {
    Quo, Rem int
}
```

We will have two functions, multiply and divide to be callable on the RPC server. These functions will need to be registered with the RPC system. The function `Register` takes a single parameter, which is an interface. So we need a type with these two functions:

```go
type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
	*reply = args.A * args.B
	return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
	if args.B == 0 {
		return error.String("divide by zero")
	}
	quo.Quo = args.A / args.B
	quo.Rem = args.A % args.B
	return nil
}
```

The underlying type of `Arith` is given as `int`. That doesn't matter - any type could have done.

An object of this type can now be registered using `Register`, and then its methods can be called by the RPC system.

### HTTP RPC Server

Any RPC needs a transport mechanism to get messages across the network. Go can use HTTP or TCP. The advantage of the HTTP mechanism is that it can leverage off the HTTP suport library. You need to add an RPC handler to the HTTP layer which is done using `HandleHTTP` and then start an HTTP server. The complete code is

```go
/**
* ArithServer
 */

package main

import (
	"fmt"
	"net/rpc"
	"errors"
	"net/http"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
	*reply = args.A * args.B
	return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
	if args.B == 0 {
		return errors.New("divide by zero")
	}
	quo.Quo = args.A / args.B
	quo.Rem = args.A % args.B
	return nil
}

func main() {

	arith := new(Arith)
	rpc.Register(arith)
	rpc.HandleHTTP()

	err := http.ListenAndServe(":1234", nil)
	if err != nil {
		fmt.Println(err.Error())
	}
}
```

### HTTP RPC client

The client needs to set up an HTTP connection to the RPC server. It needs to prepare a structure with the values to be sent, and the address of a variable to store the results in. Then it can make a `Call` with arguments:

* The name of the remote function to execute
* The values to be sent
* The address of a variable to store the result in

A client that calls both functions of the arithmetic server is

```go
/**
* ArithClient
 */

package main

import (
	"net/rpc"
	"fmt"
	"log"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "server")
		os.Exit(1)
	}
	serverAddress := os.Args[1]

	client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}
	// Synchronous call
	args := Args{17, 8}
	var reply int
	err = client.Call("Arith.Multiply", args, &reply)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

	var quot Quotient
	err = client.Call("Arith.Divide", args, &quot)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

### TCP RPC server

A version of the server that uses TCP sockets is

```go
/**
* TCPArithServer
 */

package main

import (
	"fmt"
	"net/rpc"
	"errors"
	"net"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
	*reply = args.A * args.B
	return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
	if args.B == 0 {
		return errors.New("divide by zero")
	}
	quo.Quo = args.A / args.B
	quo.Rem = args.A % args.B
	return nil
}

func main() {

	arith := new(Arith)
	rpc.Register(arith)

	tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	/* This works:
	rpc.Accept(listener)
	*/
	/* and so does this:
	 */
	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		rpc.ServeConn(conn)
	}

}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

Note that the call to `Accept` is blocking, and just handles client connections. If the server wishes to do other work as well, it should call this in a goroutine.

### TCP RPC client

A client that uses the TCP server and calls both functions of the arithmetic server is

```go
/**
* TCPArithClient
 */

package main

import (
	"net/rpc"
	"fmt"
	"log"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "server:port")
		os.Exit(1)
	}
	service := os.Args[1]

	client, err := rpc.Dial("tcp", service)
	if err != nil {
		log.Fatal("dialing:", err)
	}
	// Synchronous call
	args := Args{17, 8}
	var reply int
	err = client.Call("Arith.Multiply", args, &reply)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

	var quot Quotient
	err = client.Call("Arith.Divide", args, &quot)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

### Matching values

We note that the types of the value arguments are not the same on the client and server. In the server, we have used `Values` while in the client we used `Args`. That doesn't matter, as we are following the rules of `gob` serialisation, and the names an types of the two structures' fields match. Better programming practise would say that the names should be the same!

However, this does point out a possible trap in using Go RPC. If we change the structure in the client to be, say,

```go
type Values struct {
	C, B int
}
```
    
then `gob` has no problems: on the server-side the unmarshalling will ignore the value of C given by the client, and use the default zero value for A.

Using Go RPC will require a rigid enforcement of the stability of field names and types by the programmer. We note that there is no version control mechanism to do this, and no mechanism in `gob` to signal any possible mismatches. 

