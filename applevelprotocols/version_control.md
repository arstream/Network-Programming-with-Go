## Version control

A protocol used in a client/server system will evolve over time, changing as the system expands. This raises compatibility problems: a version 2 client will make requests that a version 1 server doesn't understand, whereas a version 2 server will send replies that a version 1 client won't understand.

Each side should ideally be able to understand messages for its own version and all earlier ones. It should be able to write replies to old style queries in old style response format. 

![version](../assets/version.png)

The ability to talk earlier version formats may be lost if the protocol changes too much. In this case, you need to be able to ensure that no copies of the earlier version still exist - and that is generally impossible.

Part of the protocol setup should involve version information.

### The Web

The Web is a good example of a system that is messed up by different versions. The protocol has been through three versions, and most servers/browsers now use the latest version. The version is given in each request 

request 	        | version
                   -|-
GET / 	            | pre 1.0
GET /  HTTP/1.0 	| HTTP 1.0
GET /  HTTP/1.1 	| HTTP 1.1 

 But the *content* of the messages has been through a large number of versions:

* HTML versions 1-4 (all different), with version 5 on the horizon;
* non-standard tags recognised by different browsers;
* non-HTML documents often require content handlers that may or may not be present - does your browser have a handler for Flash?
* inconsistent treatment of document content (e.g. some stylesheet content will crash some browsers)
* Different support for JavaScript (and different versions of JavaScript)
* Different runtime engines for Java
* Many pages do not conform to *any* HTML versions (e.g. with syntax errors)

