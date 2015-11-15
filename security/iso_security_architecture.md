## ISO security architecture

The ISO OSI (open systems interconnect) seven-layer model of distributed systems is well known and is repeated in this figure: 

![iso](../assets/iso.gif)

What is less well known is that ISO built a whole series of documents upon this architecture. For our purposes here, the most important is the ISO Security Architecture model, ISO 7498-2.

### Functions and levels

The principal functions required of a security system are

* Authentication - proof of identity
* Data integrity - data is not tampered with
* Confidentiality - data is not exposed to others
* Notarization/signature
* Access control
* Assurance/availability

These are required at the following levels of the OSI stack:

* Peer entity authentication (3, 4, 7)
* Data origin authentication (3, 4, 7)
* Access control service (3, 4, 7)
* Connection confidentiality (1, 2, 3, 4, 6, 7)
* Connectionless confidentiality (1, 2, 3, 4, 6, 7)
* Selective field confidentiality (6, 7)
* Traffic flow confidentiality (1, 3, 7)
* Connection integrity with recovery (4, 7)
* Connection integrity without recovery (4, 7)
* Connection integrity selective field (7)
* Connectionless integrity selective field (7)
* Non-repudiation at origin (7)
* Non-repudiation of receipt (7)

### Mechanisms

* Peer entity authentication
    *     encryption
    *     digital signature
    *     authentication exchange 
* Data origin authentication
    *     encryption
    *     digital signature 
* Access control service
    *     access control lists
    *     passwords
    *     capabilities lists
    *     labels
* Connection confidentiality
    *     encryption
    *     routing control 
* Connectionless confidelity
    *     encryption
    *     routing control 
* Selective field confidelity
    *     encryption 
* Traffic flow confidelity
    *     encryption
    *     traffic padding
    *     routing control 
* Connection integrity with recovery
    *     encryption
    *     data integrity 
* Connection integrity without recovery
    *     encryption
    *     data integrity 
* Connection integrity selective field
    *     encryption
    *     data integrity 
* Connectionless integrity
    *     encryption
    *     digital signature
    *     data integrity 
* Connectionless integrity selective field
    *     encryption
    *     digital signature
    *     data integrity 
* Non-repudiation at origin
    *     digital signature
    *     data integrity
    *     notarization 
* Non-repudiation of receipt
    *     digital signature
    *     data integrity
    *     notarization 
