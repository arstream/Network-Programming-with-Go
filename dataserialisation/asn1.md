# ASN.1

 ASN.1

Abstract Syntax Notation One (ASN.1) was originally designed in 1984 for the telecommunications industry. ASN.1 is a complex standard, and a subset of it is supported by Go in the package "asn1". It builds self-describing serialised data from complex data structures. Its primary use in current networking systems is as the encoding for X.509 certificates which are heavily used in authentication systems. The support in Go is based on what is needed to read and write X.509 certificates.

Two functions allow us to marshal and unmarshal data

```go
func Marshal(val interface{}) ([]byte, os.Error)
func Unmarshal(val interface{}, b []byte) (rest []byte, err os.Error)
```    

The first marshals a data value into a serialised byte array, and the second unmarshals it. However, the first argument of type `interface` deserves further examination. Given a variable of a type, we can marshal it by just passing its value. To unmarshal it, we need a variable of a named type that will match the serialised data. The precise details of this are discussed later. But we also need to make sure that the variable is allocated to memory for that type, so that there is actually existing memory for the unmarshalling to write values into.

We illustrate with an almost trivial example, of marshalling and unmarshalling an integer. We can pass an integer value to `Marshal` to return a byte array, and unmarshal the array into an integer variable as in this program:

```go
/* ASN.1
 */

package main

import (
	"encoding/asn1"
	"fmt"
	"os"
)

func main() {
	mdata, err := asn1.Marshal(13)
	checkError(err)

	var n int
	_, err1 := asn1.Unmarshal(mdata, &n)
	checkError(err1)

	fmt.Println("After marshal/unmarshal: ", n)
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
```

The unmarshalled value, is of course, 13.

Once we move beyond this, things get harder. In order to manage more complex data types, we have to look more closely at the data structures supported by ASN.1, and how ASN.1 support is done in Go.

Any serialisation method will be able to handle certain data types and not handle some others. So in order to determine the suitability of any serialisation such as ASN.1, you have to look at the possible data types supported versus those you wish to use in your application. The following ASN.1 types are taken from http://www.obj-sys.com/asn1tutorial/node4.html

The simple types are

* BOOLEAN: two-state variable values
* INTEGER: Model integer variable values
* BIT STRING: Model binary data of arbitrary length
* OCTET STRING: Model binary data whose length is a multiple of eight
* NULL: Indicate effective absence of a sequence element
* OBJECT IDENTIFIER: Name information objects
* REAL: Model real variable values
* ENUMERATED: Model values of variables with at least three states
* CHARACTER STRING: Models values that are strings of characters

Character strings can be from certain character sets

* NumericString: 0,1,2,3,4,5,6,7,8,9, and space
* PrintableString: Upper and lower case letters, digits, space, apostrophe, left/right parenthesis, plus sign, comma, hyphen, full stop, solidus, colon, equal sign, question mark
* TeletexString (T61String): The Teletex character set in CCITT's T61, space, and delete
* VideotexString: The Videotex character set in CCITT's T.100 and T.101, space, and delete
* VisibleString (ISO646String): Printing character sets of international ASCII, and space
* IA5String: International Alphabet 5 (International ASCII)
* GraphicString 25 All registered G sets, and space GraphicString

And finally, there are the structured types:

* SEQUENCE: Models an ordered collection of variables of different type
* SEQUENCE OF: Models an ordered collection of variables of the same type
* SET: Model an unordered collection of variables of different types
* SET OF: Model an unordered collection of variables of the same type
* CHOICE: Specify a collection of distinct types from which to choose one type
* SELECTION: Select a component type from a specified CHOICE type
* ANY: Enable an application to specify the type Note: ANY is a deprecated ASN.1 * Structured Type. It has been replaced with X.680 Open Type.

Not all of these are supported by Go. Not all possible values are supported by Go. The rules as given in the Go "asn1" package documentation are

* An ASN.1 INTEGER can be written to an `int` or `int64`. If the encoded value does not fit in the Go type, Unmarshal returns a parse error.
* An ASN.1 BIT STRING can be written to a BitString.
* An ASN.1 OCTET STRING can be written to a `[]byte`.
* An ASN.1 OBJECT IDENTIFIER can be written to an ObjectIdentifier.
* An ASN.1 ENUMERATED can be written to an Enumerated.
* An ASN.1 UTCTIME or GENERALIZEDTIME can be written to a `*time.Time`.
* An ASN.1 PrintableString or IA5String can be written to a string.
* Any of the above ASN.1 values can be written to an `interface{}`. The value stored in the interface has the corresponding Go type. For integers, that type is `int64`.
* An ASN.1 SEQUENCE OF x or SET OF x can be written to a slice if an x can be written to * the slice's element type.
* An ASN.1 SEQUENCE or SET can be written to a struct if each of the elements in the * sequence can be written to the corresponding element in the struct.

Go places real restrictions on ASN.1. For example, ASN.1 allows integers of any size, while the Go implementation will only allow upto signed 64-bit integers. On the other hand, Go distinguishes between signed and unsigned types, while ASN.1 doesn't. So for example, transmitting a value of `uint64` may fail if it is too large for `int64`.

In a similar vein, ASN.1 allows several different character sets. Go only supports PrintableString and IA5String (ASCII). ASN.1 does not support Unicode characters (which require the BMPString ASN.1 extension). The basic Unicode character set of Go is not supported, and if an application requires transport of Unicode characters, then an encoding such as UTF-7 will be needed. Such encodings are discussed in a later chapter on character sets.

We have seen that a value such as an integer can be easily marshalled and unmarshalled. Other basic types such as booleans and reals can be similarly dealt with. Strings which are composed entirely of ASCII characters can be marshalled and unmarshalled. However, if the string is, for example, `"hello \u00bc"` which contains the non-ASCII character `'¼'` then an error will occur: `"ASN.1 structure error: PrintableString contains invalid character"`. This code works, as long as the string is only composed of printable characters:

```go
s := "hello"
mdata, _ := asn1.Marshal(s)

var newstr string
asn1.Unmarshal(mdata, &newstr)
```

ASN.1 also includes some "useful types" not in the above list, such as UTC time. Go supports this UTC time type. This means that you can pass time values in a way that is not possible for other data values. ASN.1 does not support pointers, but Go has special code to manage pointers to time values. The function `GetLocalTime` returns `*time.Time`. The special code marshals this, and it can be unmarshalled into a pointer variable to a `time.Time` object. Thus this code works

```go
t := time.LocalTime()
mdata, err := asn1.Marshal(t)

var newtime = new(time.Time)
_, err1 := asn1.Unmarshal(&newtime, mdata)
```    

Both `LocalTime` and `new` handle pointers to a `*time.Time`, and Go looks after this special case.

In general, you will probably want to marshal and unmarshal structures. Apart from the special case of time, Go will happily deal with structures, but not with pointers to structures. Operations such as `new` create pointers, so you have to dereference them before marshalling/unmarshalling them. Go normally dereferences pointers for you when needed, but not in this case. These both work for a type `T`:

```go
// using variables
var t1 T
t1 = ...
mdata1, _ := asn1.Marshal(t)

var newT1 T
asn1.Unmarshal(&newT1, mdata1)

/// using pointers
var t2 = new(T)
*t2 = ...
mdata2, _ := asn1.Marshal(*t2)

var newT2 = new(T)
asn1.Unmarshal(newT2, mdata2)
```

Any suitable mix of pointers and variables will work as well.

The fields of a structure must all be exportable, that is, field names must begin with an uppercase letter. Go uses the `reflect` package to marshal/unmarshal structures, so it must be able to examine all fields. This type cannot be marshalled:

```go
type T struct {
  Field1 int
  field2 int // not exportable
}
```

ASN.1 only deals with the data types. It does not consider the names of structure fields. So the following type `T1` can be marshalled/unmarshalled into type `T2` as the corresponding fields are the same types:

```go
type T1 struct {
    F1 int
    F2 string
}

type T2 struct {
    FF1 int
    FF2 string
}
```

Not only the types of each field must match, but the number must match as well. These two types don't work:

```go
type T1 struct {
    F1 int
}

type T2 struct {
    F1 int
    F2 string // too many fields
}
```

### ASN.1 daytime client and server

Now (finally) let us turn to using ASN.1 to transport data across the network.

We can write a TCP server that delivers the current time as an ASN.1 Time type, using the techniques of the last chapter. A server is

```go
/* ASN1 DaytimeServer
 */
package main

import (
	"encoding/asn1"
	"fmt"
	"net"
	"os"
	"time"
)

func main() {

	service := ":1200"
	tcpAddr, err := net.ResolveTCPAddr("tcp", service)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}

		daytime := time.Now()
		// Ignore return network errors.
		mdata, _ := asn1.Marshal(daytime)
		conn.Write(mdata)
		conn.Close() // we're finished
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}
```

which can be compiled to an executable such as `ASN1DaytimeServer` and run with no arguments. It will wait for connections and then send the time as an ASN.1 string to the client.

A client is

```go
/* ASN.1 DaytimeClient
 */
package main

import (
	"bytes"
	"encoding/asn1"
	"fmt"
	"io"
	"net"
	"os"
	"time"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
		os.Exit(1)
	}
	service := os.Args[1]

	conn, err := net.Dial("tcp", service)
	checkError(err)

	result, err := readFully(conn)
	checkError(err)

	var newtime time.Time
	_, err1 := asn1.Unmarshal(result, &newtime)
	checkError(err1)

	fmt.Println("After marshal/unmarshal: ", newtime.String())

	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}

func readFully(conn net.Conn) ([]byte, error) {
	defer conn.Close()

	result := bytes.NewBuffer(nil)
	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		result.Write(buf[0:n])
		if err != nil {
			if err == io.EOF {
				break
			}
			return nil, err
		}
	}
	return result.Bytes(), nil
}
```

This connects to the service given in a form such as `localhost:1200`, reads the TCP packet and decodes the ASN.1 content back into a string, which it prints.

We should note that neither of these two - the client or the server - are compatable with the text-based clients and servers of the last chapter. 
This client and server are exchanging ASN.1 encoded data values, not textual strings. 