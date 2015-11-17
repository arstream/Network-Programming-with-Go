## Configuring HTTP requests

Go also supplies a lower-level interface for user agents to communicate with HTTP servers. As you might expect, not only does it give you more control over the client requests, but requires you to spend more effort in building the requests. However, there is only a small increase.

The data type used to build requests is the type `Request`. This is a complex type, and is given in the Go documentation as

```go
type Request struct {
    Method     string // GET, POST, PUT, etc.
    RawURL     string // The raw URL given in the request.
    URL        *URL   // Parsed URL.
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0


    // A header maps request lines to their values.
    // If the header says
    //
    //	accept-encoding: gzip, deflate
    //	Accept-Language: en-us
    //	Connection: keep-alive
    //
    // then
    //
    //	Header = map[string]string{
    //		"Accept-Encoding": "gzip, deflate",
    //		"Accept-Language": "en-us",
    //		"Connection": "keep-alive",
    //	}
    //
    // HTTP defines that header names are case-insensitive.
    // The request parser implements this by canonicalizing the
    // name, making the first character and any characters
    // following a hyphen uppercase and the rest lowercase.
    Header map[string]string

    // The message body.
    Body io.ReadCloser

    // ContentLength records the length of the associated content.
    // The value -1 indicates that the length is unknown.
    // Values >= 0 indicate that the given number of bytes may be read from Body.
    ContentLength int64

    // TransferEncoding lists the transfer encodings from outermost to innermost.
    // An empty list denotes the "identity" encoding.
    TransferEncoding []string

    // Whether to close the connection after replying to this request.
    Close bool

    // The host on which the URL is sought.
    // Per RFC 2616, this is either the value of the Host: header
    // or the host name given in the URL itself.
    Host string

    // The referring URL, if sent in the request.
    //
    // Referer is misspelled as in the request itself,
    // a mistake from the earliest days of HTTP.
    // This value can also be fetched from the Header map
    // as Header["Referer"]; the benefit of making it
    // available as a structure field is that the compiler
    // can diagnose programs that use the alternate
    // (correct English) spelling req.Referrer but cannot
    // diagnose programs that use Header["Referrer"].
    Referer string

    // The User-Agent: header string, if sent in the request.
    UserAgent string

    // The parsed form. Only available after ParseForm is called.
    Form map[string][]string

    // Trailer maps trailer keys to values.  Like for Header, if the
    // response has multiple trailer lines with the same key, they will be
    // concatenated, delimited by commas.
    Trailer map[string]string
}
```

There is a lot of information that can be stored in a request. You do not need to fill in all fields, only those of interest. The simplest way to create a request with default values is by for example

``` 
request, err := http.NewRequest("GET", url.String(), nil)
``` 
  
Once a request has been created, you can modify fields. For example, to specify that you only wish to receive UTF-8, add an "Accept-Charset" field to a request by

```  
request.Header.Add("Accept-Charset", "UTF-8;q=1, ISO-8859-1;q=0")
``` 
  
(Note that the default set ISO-8859-1 always gets a value of one unless mentioned explicitly in the list.).

A client setting a charset request is simple by the above. But there is some confusion about what happens with the server's return value of a charset. The returned resource *should* have a `Content-Type` which will specify the media type of the content such as `text/html`. If appropriate the media type should state the charset, such as `text/html; charset=UTF-8`. If there is no charset specification, then according to the HTTP specification it should be treated as the default ISO8859-1 charset. But the HTML 4 specification states that since many servers don't conform to this, then you can't make any assumptions.

If there is a charset specified in the server's `Content-Type`, then assume it is correct. if there is none specified, since 50% of pages are in UTF-8 and 20% are in ASCII then it is safe to assume UTF-8. Only 30% of pages may be wrong :-(. 