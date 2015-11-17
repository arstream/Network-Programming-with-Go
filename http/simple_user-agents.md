## Simple user-agents

User agents such as browsers make requests and get responses. The response type is

```go
type Response struct {
    Status     string // e.g. "200 OK"
    StatusCode int    // e.g. 200
    Proto      string // e.g. "HTTP/1.0"
    ProtoMajor int    // e.g. 1
    ProtoMinor int    // e.g. 0

    RequestMethod string // e.g. "HEAD", "CONNECT", "GET", etc.

    Header map[string]string

    Body io.ReadCloser

    ContentLength int64

    TransferEncoding []string

    Close bool

    Trailer map[string]string
}
```  

We shall examine this data structure through examples. The simplest request is from a user agent is "HEAD" which asks for information about a resource and its HTTP server. The function

```go
func Head(url string) (r *Response, err os.Error)
```

can be used to make this query.

The status of the response is in the response field `Status`, while the field `Header` is a map of the header fields in the HTTP response. A program to make this request and display the results is

```go
/* Head
 */

package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "host:port")
		os.Exit(1)
	}
	url := os.Args[1]

	response, err := http.Head(url)
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(2)
	}

	fmt.Println(response.Status)
	for k, v := range response.Header {
		fmt.Println(k+":", v)
	}

	os.Exit(0)
}
```

When run against a resource as in `Head http://www.golang.com/` it prints something like

```
200 OK
Content-Type: text/html; charset=utf-8
Date: Tue, 14 Sep 2015 05:34:29 GMT
Cache-Control: public, max-age=3600
Expires: Tue, 14 Sep 2015 06:34:29 GMT
Server: Google Frontend
``` 

Usually, we are want to retrieve a resource rather than just get information about it. The "GET" request will do this, and this can be done using

```go
func Get(url string) (r *Response, finalURL string, err os.Error)
```    

The content of the response is in the response field `Body` which is of type `io.ReadCloser`. We can print the content to the screen with the following program

```go
/* Get
 */

package main

import (
	"fmt"
	"net/http"
	"net/http/httputil"
	"os"
	"strings"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "host:port")
		os.Exit(1)
	}
	url := os.Args[1]

	response, err := http.Get(url)
	if err != nil {
		fmt.Println(err.Error())
		os.Exit(2)
	}

	if response.Status != "200 OK" {
		fmt.Println(response.Status)
		os.Exit(2)
	}

	b, _ := httputil.DumpResponse(response, false)
	fmt.Print(string(b))

	contentTypes := response.Header["Content-Type"]
	if !acceptableCharset(contentTypes) {
		fmt.Println("Cannot handle", contentTypes)
		os.Exit(4)
	}

	var buf [512]byte
	reader := response.Body
	for {
		n, err := reader.Read(buf[0:])
		if err != nil {
			os.Exit(0)
		}
		fmt.Print(string(buf[0:n]))
	}
	os.Exit(0)
}

func acceptableCharset(contentTypes []string) bool {
	// each type is like [text/html; charset=UTF-8]
	// we want the UTF-8 only
	for _, cType := range contentTypes {
		if strings.Index(cType, "UTF-8") != -1 {
			return true
		}
	}
	return false
}
```

Note that there are important character set issues of the type discussed in the previous chapter. The server will deliver the content using some character set encoding, and possibly some transfer encoding. Usually this is a matter of negotiation between user agent and server, but the simple `Get` command that we are using does not include the user agent component of the negotiation. So the server can send whatever character encoding it wishes.

At the time of first writing, I was in China. When I tried this program on `www.google.com`, Google's server tried to be helpful by guessing my location and sending me the text in the Chinese character set Big5! How to tell the server what character encoding is okay for me is discussed later. 

