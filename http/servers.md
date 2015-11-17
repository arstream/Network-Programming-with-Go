## Servers

The other side to building a client is a Web server handling HTTP requests. The simplest - and earliest - servers just returned copies of files. However, any URL can now trigger an arbitrary computation in current servers.

### File server

We start with a basic file server. Go supplies a *multi-plexer*, that is, an object that will read and interpret requests. It hands out requests to *handlers* which run in their own thread. Thus much of the work of reading HTTP requests, decoding them and branching to suitable functions in their own thread is done for us.

For a file server, Go also gives a `FileServer` object which knows how to deliver files from the local file system. It takes a "root" directory which is the top of a file tree in the local system, and a pattern to match URLs against. The simplest pattern is "/" which is the top of any URL. This will match all URLs.

An HTTP server delivering files from the local file system is almost embarrassingly trivial given these objects. It is

```go
/* File Server
 */

package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	// deliver files from the directory /var/www 
	//fileServer := http.FileServer(http.Dir("/var/www"))
	fileServer := http.FileServer(http.Dir("/home/httpd/html/"))

	// register the handler and deliver requests to it
	err := http.ListenAndServe(":8000", fileServer)
	checkError(err)
	// That's it!
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

This server even delivers `"404 not found"` messages for requests for file resources that don't exist!

### Handler functions

In this last program, the handler was given in the second argument to `ListenAndServe`. Any number of handlers can be registered first by calls to `Handle` or `HandleFunc`, with signatures

```go
func Handle(pattern string, handler Handler)
func HandleFunc(pattern string, handler func(*Conn, *Request))
```

The second argument to `HandleAndServe` could be `nil`, and then calls are dispatched to all registered handlers. Each handler should have a different URL pattern. For example, the file handler might have URL pattern "/" while a function handler might have URL pattern "/cgi-bin". A more specific pattern takes precedence over a more general pattern.

Common CGI programs are `test-cgi` (written in the shell) or `printenv` (written in Perl) which print the values of the environment variables. A handler can be written to work in a similar manner.

```go
/* Print Env
 */

package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	// file handler for most files
	fileServer := http.FileServer(http.Dir("/var/www"))
	http.Handle("/", fileServer)

	// function handler for /cgi-bin/printenv
	http.HandleFunc("/cgi-bin/printenv", printEnv)

	// deliver requests to the handlers
	err := http.ListenAndServe(":8000", nil)
	checkError(err)
	// That's it!
}

func printEnv(writer http.ResponseWriter, req *http.Request) {
	env := os.Environ()
	writer.Write([]byte("<h1>Environment</h1>\n<pre>"))
	for _, v := range env {
		writer.Write([]byte(v + "\n"))
	}
	writer.Write([]byte("</pre>"))
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

*Note: for simplicity this program does not deliver well-formed HTML. It is missing html, head and body tags.*

Using the `cgi-bin` directory in this program is a bit cheeky: it doesn't call an external program like CGI scripts do. It just calls a Go function. Go does have the ability to call external programs using `os.ForkExec`, but does not yet have support for dynamically linkable modules like Apache's `mod_perl`. 


### Bypassing the default multiplexer

HTTP requests received by a Go server are usually handled by a multiplexer the examines the path in the HTTP request and calls the appropriate file handler, etc. You can define your own handlers. These can either be registered with the default multiplexer by calling `http.HandleFunc` which takes a pattern and a function. The functions such as `ListenAndServe` then take a `nil` handler function. This was done in the last example.

If you want to take over the multiplexer role then you can give a non-zero function as the handler function. This function will then be totally responsible for managing the requests and responses.

The following example is trivial, but illustrates the use of this: the multiplexer function simply returns a `"204 No content"` for all requests:

```go
/* ServerHandler
 */

package main

import (
	"net/http"
)

func main() {

	myHandler := http.HandlerFunc(func(rw http.ResponseWriter, request *http.Request) {
		// Just return no content - arbitrary headers can be set, arbitrary body
		rw.WriteHeader(http.StatusNoContent)
	})

	http.ListenAndServe(":8080", myHandler)
}
```

Arbitrarily complex behaviour can be built, of course. 

### Low-level servers

Go also supplies a lower-level interface for servers. Again, this means that as the programmer you have to do more work. You first make a TCP server, and then wrap a `ServerConn` around it. Then you read `Request`'s and write `Response`'s.

## Conclusion

Go has extensive support for HTTP. This is not surprising, since Go was partly invented to fill a need by Google for their own servers.
