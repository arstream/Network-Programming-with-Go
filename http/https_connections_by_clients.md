## HTTPS connections by clients

For secure, encrypted connections, HTTP uses TLS which is described in the chapter on security. The protocol of HTTP+TLS is called HTTPS and uses `https://` urls instead of `http://` urls.

Servers are required to return valid X.509 certificates before a client will accept data from them. If the certificate is valid, then Go handles everything under the hood and the clients given previously run okay with https URLs.

Many sites have invalid certificates. They may have expired, they may be self-signed instead of by a recognised Certificate Authority or they may just have errors (such as having an incorrect server name). Browsers such as Firefox put a big warning notice with a "Get me out of here!" button, but you can carry on at your risk - which many people do.

Go presently bails out when it encounters certificate errors. There is cautious support for carrying on but I haven't got it working yet. So there is no current example for "carrying on in the face of adversity :-)". Maybe later. 

