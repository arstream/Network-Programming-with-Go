## Overview of HTTP

### URLs and resources

URLs specify the location of a *resource*. A resource is often a static file, such as an HTML document, an image, or a sound file. But increasingly, it may be a dynamically generated object, perhaps based on information stored in a database.

When a user agent requests a resource, what is returned is not the resource itself, but some *representation* of that resource. For example, if the resource is a static file, then what is sent to the user agent is a copy of the file.

Multiple URLs may point to the same resource, and an HTTP server will return appropriate representations of the resource for each URL. For example, an company might make product information available both internally and externally using different URLs for the same product. The internal representation of the product might include information such as internal contact officers for the product, while the external representation might include the location of stores selling the product.

This view of resources means that the HTTP protocol can be fairly simple and straightforward, while an HTTP server can be arbitrarily complex. HTTP has to deliver requests from user agents to servers and return a byte stream, while a server might have to do any amount of processing of the request.

### HTTP characteristics

HTTP is a stateless, connectionless, reliable protocol. In the simplest form, each request from a user agent is handled reliably and then the connection is broken. Each request involves a separate TCP connection, so if many resources are required (such as images embedded in an HTML page) then many TCP connections have to be set up and torn down in a short space of time.

Thera are many optimisations in HTTP which add complexity to the simple structure, in order to create a more efficient and reliable protocol.

### Versions

There are 3 versions of HTTP

* Version 0.9 - totally obsolete
* Version 1.0 - almost obsolete
* Version 1.1 - current

Each version must understand requests and responses of earlier versions.

### HTTP 0.9

#### Request format

```
Request = Simple-Request

Simple-Request = "GET" SP Request-URI CRLF
```


#### Response format

A response is of the form

```
Response = Simple-Response

Simple-Response = [Entity-Body]
```

### HTTP 1.0

This version added much more information to the requests and responses. Rather than "grow" the 0.9 format, it was just left alongside the new version.

#### Request format

The format of requests from client to server is

```
Request = Simple-Request | Full-Request

Simple-Request = "GET" SP Request-URI CRLF

Full-Request = Request-Line
		*(General-Header
		| Request-Header
		| Entity-Header)
		CRLF
		[Entity-Body]
```

A Simple-Request is an HTTP/0.9 request and must be replied to by a Simple-Response.

A Request-Line has format

```
Request-Line = Method SP Request-URI SP HTTP-Version CRLF
```

where

```
Method = "GET" | "HEAD" | POST |
	 extension-method
```

e.g. `GET http://jan.newmarch.name/index.html HTTP/1.0`

#### Response format

A response is of the form

```
Response = Simple-Response | Full-Response

Simple-Response = [Entity-Body]

Full-Response = Status-Line
		*(General-Header 
		| Response-Header
		| Entity-Header)
		CRLF
		[Entity-Body]
```

The Status-Line gives information about the fate of the request:
```
Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
```
e.g.
```
HTTP/1.0 200 OK

The codes are

Status-Code =	  "200" ; OK
		| "201" ; Created
		| "202" ; Accepted
		| "204" ; No Content
		| "301" ; Moved permanently
		| "302" ; Moved temporarily
		| "304" ; Not modified
		| "400" ; Bad request
		| "401" ; Unauthorised
		| "403" ; Forbidden
		| "404" ; Not found
		| "500" ; Internal server error
		| "501" ; Not implemented
		| "502" ; Bad gateway
		| "503" | Service unavailable
		| extension-code
```

The Entity-Header contains useful information about the Entity-Body to follow

```
Entity-Header =	Allow
		| Content-Encoding
		| Content-Length
		| Content-Type
		| Expires
		| Last-Modified
		| extension-header
```

For example

```
HTTP/1.1 200 OK
Date: Fri, 29 Aug 2003 00:59:56 GMT
Server: Apache/2.0.40 (Unix)
Accept-Ranges: bytes
Content-Length: 1595
Connection: close
Content-Type: text/html; charset=ISO-8859-1
```

### HTTP 1.1

HTTP 1.1 fixes many problems with HTTP 1.0, but is more complex because of it. This version is done by extending or refining the options available to HTTP 1.0. e.g.

* there are more commands such as TRACE and CONNECT
* you should use absolute URLs, particularly for connecting by proxies e.g

```
GET http://www.w3.org/index.html HTTP/1.1
```


* there are more attributes such as If-Modified-Since, also for use by proxies

The changes include

* hostname identification (allows virtual hosts)
* content negotiation (multiple languages)
* persistent connections (reduces TCP overheads - this is very messy)
* chunked transfers
* byte ranges (request parts of documents)
* proxy support 

The 0.9 protocol took one page. The 1.0 protocol was described in about 20 pages. 1.1 takes 120 pages. 