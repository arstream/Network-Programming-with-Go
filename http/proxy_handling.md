## Proxy handling

### Simple proxy

HTTP 1.1 laid out how HTTP should work through a proxy. A "GET" request should be made to a proxy. However, the URL requested should be the full URL of the destination. In addition the HTTP header should contain a "Host" field, set to the proxy. As long as the proxy is configured to pass such requests through, then that is all that needs to be done.

Go considers this to be part of the HTTP transport layer. To manage this it has a class `Transport`. This contains a field which can be set to a *function* that returns a URL for a proxy. If we have a URL as a string for the proxy, the appropriate transport object is created and then given to a client object by

```
proxyURL, err := url.Parse(proxyString)
transport := &http.Transport{Proxy: http.ProxyURL(proxyURL)}
client := &http.Client{Transport: transport}
``` 
  
The client can then continue as before.

The following program illustrates this:

```go
/* ProxyGet
 */

package main

import (
	"fmt"
	"io"
	"net/http"
	"net/http/httputil"
	"net/url"
	"os"
)

func main() {
	if len(os.Args) != 3 {
		fmt.Println("Usage: ", os.Args[0], "http://proxy-host:port http://host:port/page")
		os.Exit(1)
	}
	proxyString := os.Args[1]
	proxyURL, err := url.Parse(proxyString)
	checkError(err)
	rawURL := os.Args[2]
	url, err := url.Parse(rawURL)
	checkError(err)

	transport := &http.Transport{Proxy: http.ProxyURL(proxyURL)}
	client := &http.Client{Transport: transport}

	request, err := http.NewRequest("GET", url.String(), nil)

	dump, _ := httputil.DumpRequest(request, false)
	fmt.Println(string(dump))

	response, err := client.Do(request)

	checkError(err)
	fmt.Println("Read ok")

	if response.Status != "200 OK" {
		fmt.Println(response.Status)
		os.Exit(2)
	}
	fmt.Println("Reponse ok")

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

func checkError(err error) {
	if err != nil {
		if err == io.EOF {
			return
		}
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

If you have a proxy at, say, XYZ.com on port 8080, test this by

``` 
go run ProxyGet.go http://XYZ.com:8080/ http://www.google.com
``` 
  
If you don't have a suitable proxy to test this, then download and install the Squid proxy to your own computer.

The above program used a known proxy passed as an argument to the program. There are many ways in which proxies can be made known to applications. Most browsers have a configuration menu in which you can enter proxy information: such information is not available to a Go application. Some applications may get proxy information from an `autoproxy.pac` file somewhere in your network: Go does not (yet) know how to parse these JavaScript files and so cannot use them. Linux systems using Gnome have a configuration system called `gconf` in which proxy information can be stored: Go cannot access this. *But* it can find proxy information if it is set in operating system environment variables such as HTTP_PROXY or http_proxy using the function

```go    
func ProxyFromEnvironment(req *Request) (*url.URL, error)
```    
  
If your programs are running in such an environment you can use this function instead of having to explicitly know the proxy parameters.

### Authenticating proxy

Some proxies will require authentication, by a user name and password in order to pass requests. A common scheme is "basic authentication" in which the user name and password are concatenated into a string "user:password" and then BASE64 encoded. This is then given to the proxy by the HTTP request header "Proxy-Authorisation" with the flag that it is the basic authentication

The following program illustrates this, adding the Proxy-Authentication header to the previous proxy program:

```go
/* ProxyAuthGet
 */

package main

import (
	"encoding/base64"
	"fmt"
	"io"
	"net/http"
	"net/http/httputil"
	"net/url"
	"os"
)

const auth = "jannewmarch:mypassword"

func main() {
	if len(os.Args) != 3 {
		fmt.Println("Usage: ", os.Args[0], "http://proxy-host:port http://host:port/page")
		os.Exit(1)
	}
	proxy := os.Args[1]
	proxyURL, err := url.Parse(proxy)
	checkError(err)
	rawURL := os.Args[2]
	url, err := url.Parse(rawURL)
	checkError(err)

	// encode the auth
	basic := "Basic " + base64.StdEncoding.EncodeToString([]byte(auth))

	transport := &http.Transport{Proxy: http.ProxyURL(proxyURL)}
	client := &http.Client{Transport: transport}

	request, err := http.NewRequest("GET", url.String(), nil)

	request.Header.Add("Proxy-Authorization", basic)
	dump, _ := httputil.DumpRequest(request, false)
	fmt.Println(string(dump))

	// send the request
	response, err := client.Do(request)

	checkError(err)
	fmt.Println("Read ok")

	if response.Status != "200 OK" {
		fmt.Println(response.Status)
		os.Exit(2)
	}
	fmt.Println("Reponse ok")

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

func checkError(err error) {
	if err != nil {
		if err == io.EOF {
			return
		}
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

