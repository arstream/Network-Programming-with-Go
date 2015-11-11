## Encoding binary data as strings

Once upon a time, transmitting 8-bit data was problematic. It was often transmitted over noisy serial lines and could easily become corrupted. 7-bit data on the other hand could be transmitted more reliably because the 8th bit could be used as check digit. For example, in an "even parity" scheme, the check digit would be set to one or zero to make an even number of 1-s in a byte. This allows detection of errors of a single bit in each byte.

ASCII is a 7-bit character set. A number of schemes have been developed that are more sophisticated than simple parity checking, but which involve translating 8-bit binary data into 7-bit ASCII format. Essentially, the 8-bit data is stretched out in some way over the 7-bit bytes.

Binary data transmitted in HTTP responses and requests is often translated into an ASCII form. This makes it easy to inspect the HTTP messages with a simple text reader without worrying about what strange 8-bit bytes might do to your display!

One common format is Base64. Go has support for many binary-to-text formats, including Base64.

There are two principal functions to use for Base64 encoding and decoding:

```go
func NewEncoder(enc *Encoding, w io.Writer) io.WriteCloser
func NewDecoder(enc *Encoding, r io.Reader) io.Reader
```

A simple program just to encode and decode a set of eight binary digits is

```go
/**
 * Base64
 */

package main

import (
	"bytes"
	"encoding/base64"
	"fmt"
)

func main() {

	eightBitData := []byte{1, 2, 3, 4, 5, 6, 7, 8}
	bb := &bytes.Buffer{}
	encoder := base64.NewEncoder(base64.StdEncoding, bb)
	encoder.Write(eightBitData)
	encoder.Close()
	fmt.Println(bb)

	dbuf := make([]byte, 12)
	decoder := base64.NewDecoder(base64.StdEncoding, bb)
	decoder.Read(dbuf)
	for _, ch := range dbuf {
		fmt.Print(ch)
	}
}
```
