## Data integrity

 Ensuring data integrity means supplying a means of testing that the data has not been tampered with. Usually this is done by forming a simple number out of the bytes in the data. This process is called *hashing* and the resulting number is called a *hash* or *hash value*.

A naive hashing algorithm is just to sum up all the bytes in the data. However, this still allows almost any amount of changing the data around and still preserving the hash values. For example, an attacker could just swap two bytes. This preserves the hash value, but could end up with you owing someone $65,536 instead of $256.

Hashing algorithms used for security purposes have to be "strong", so that it is very difficult for an attacker to find a different sequence of bytes with the same hash value. This makes it hard to modify the data to the attacker's purposes. Security researchers are constantly testing hash algorithms to see if they can break them - that is, find a simple way of coming up with byte sequences to match a hash value. They have devised a series of *cryptographic* hashing algorithms which are believed to be strong.

Go has support for several hashing algorithms, including MD4, MD5, RIPEMD-160, SHA1, SHA224, SHA256, SHA384 and SHA512. They all follow the same pattern as far as the Go programmer is concerned: a function `New` (or similar) in the appropriate package returns a `Hash` object from the `hash` package.

A `Hash` has an `io.Writer`, and you write the data to be hashed to this writer. You can query the number of bytes in the hash value by `Size` and the hash value by `Sum`.

A typical case is MD5 hashing. This uses the `md5` package. The hash value is a 16 byte array. This is typically printed out in ASCII form as four hexadecimal numbers, each made of 4 bytes. A simple program is 

```go
/* MD5Hash
 */

package main

import (
	"crypto/md5"
	"fmt"
)

func main() {
	hash := md5.New()
	bytes := []byte("hello\n")
	hash.Write(bytes)
	hashValue := hash.Sum(nil)
	hashSize := hash.Size()
	for n := 0; n < hashSize; n += 4 {
		var val uint32
		val = uint32(hashValue[n])<<24 +
			uint32(hashValue[n+1])<<16 +
			uint32(hashValue[n+2])<<8 +
			uint32(hashValue[n+3])
		fmt.Printf("%x ", val)
	}
	fmt.Println()
}
```

which prints `b1946ac9 2492d234 7c6235b4 d2611184`

A variation on this is the HMAC (Keyed-Hash Message Authentication Code) which adds a key to the hash algorithm. There is little change in using this. To use MD5 hashing along with a key, replace the call to `New` by

```go
func NewMD5(key []byte) hash.Hash
```