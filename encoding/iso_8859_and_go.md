## ISO 8859 and Go

The ISO 8859 series are 8-bit character sets for different parts of Europe and some other areas. They all have the ASCII set common in the low part, but differ in the top part. According to Google, ISO 8859 codes account for about 20% of the web pages it sees.

The first code, ISO 8859-1 or Latin-1, has the first 256 characters in common with Unicode. The encoded value of the Latin-1 characters is the same in UTF-16 and in the default ISO 8859-1 encoding. But this doesn't really help much, as UTF-16 is a 16-bit encoding and ISO 8859-1 is an 8-bit encoding. UTF-8 is a 8-bit encoding, but it uses the top bit to signal extra bytes, so only the ASCII subset overlaps for UTF-8 and ISO 8859-1. So UTF-8 doesn't help much either.

But the ISO 8859 series don't have any complex issues. To each character in each set corresponds a unique Unicode character. For example, in ISO 8859-2, the character "latin capital letter I with ogonek" has ISO 8859-2 code point 0xc7 (in hexadecimal) and corresponding Unicode code point of U+012E. Transforming either way between an ISO 8859 set and the corresponding Unicode characters is essentially just a table lookup.

The table from ISO 8859 code points to Unicode code points could be done as an array of 256 integers. But many of these will have the same value as the index. So we just use a map of the different ones, and those not in the map take the index value.

For ISO 8859-2 a portion of the map is

```go
var unicodeToISOMap = map[int] uint8 {
    0x12e: 0xc7,
    0x10c: 0xc8,
    0x118: 0xca,
    // plus more
}
```
    
and a function to convert UTF-8 strings to an array of ISO 8859-2 bytes is

```go
/* Turn a UTF-8 string into an ISO 8859 encoded byte array
*/ 
func unicodeStrToISO(str string) []byte {
        // get the unicode code points
	codePoints := []int(str)

        // create a byte array of the same length
	bytes := make([]byte, len(codePoints))

	for n, v := range(codePoints) {
                // see if the point is in the exception map
		iso, ok := unicodeToISOMap[v]
		if !ok {
                        // just use the value
			iso = uint8(v)
		}
		bytes[n] = iso
	}
	return bytes
}
```   

In a similar way you cacn change an array of ISO 8859-2 bytes into a UTF-8 string:

```go
var isoToUnicodeMap = map[uint8] int {
    0xc7: 0x12e, 
    0xc8: 0x10c,
    0xca: 0x118,
    // and more
}

func isoBytesToUnicode(bytes []byte) string {
	codePoints := make([]int, len(bytes))
	for n, v := range(bytes) {
		unicode, ok :=isoToUnicodeMap[v]
		if !ok {
			unicode = int(v)
		}
		codePoints[n] = unicode
	}
	return string(codePoints)
}
```

These functions can be used to read and write UTF-8 strings as ISO 8859-2 bytes. By changing the mapping table, you can cover the other ISO 8859 codes. Latin-1, or ISO 8859-1, is a special case - the exception map is empty as the code points for Latin-1 are the same in Unicode. You could also use the same technique for other character sets based on a table mapping, such as Windows 1252. 

