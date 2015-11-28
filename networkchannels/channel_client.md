## Channel client

In order to find an exported channel, the client must *import* it. This is created using `Import` which takes a protocol and a network service address of "host:port". This is then used to import a network channel by name and bind it to a local channel. Note that channel variables are *references*, so you do not need to pass their addresses to functions that change them.

The following client gets two channels to and from the echo server, and then writes and reads ten messages:

```go
/* EchoClient
 */
package main

import (
	"fmt"
	"old/netchan"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "host:port")
		os.Exit(1)
	}
	service := os.Args[1]

	importer, err := netchan.Import("tcp", service)
	checkError(err)

	fmt.Println("Got importer")
	echoIn := make(chan string)
	importer.Import("echo-in", echoIn, netchan.Recv, 1)
	fmt.Println("Imported in")

	echoOut := make(chan string)
	importer.Import("echo-out", echoOut, netchan.Send, 1)
	fmt.Println("Imported out")

	for n := 0; n < 10; n++ {
		echoOut <- "hello "
		s, ok := <-echoIn
		if !ok {
			fmt.Println("Read failure")
			break
		}
		fmt.Println(s, n)
	}
	close(echoOut)
	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```


