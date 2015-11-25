## Parsing XML

Go has an XML parser which is created using `NewParser`. This takes an `io.Reader` as parameter and returns a pointer to `Parser`. The main method of this type is `Token` which returns the next token in the input stream. The token is one of the types `StartElement`, `EndElement`, `CharData`, `Comment`, `ProcInst` or `Directive`.

The types are

`StartElement`

The type `StartElement` is a structure with two field types:

```go
type StartElement struct {
    Name Name
    Attr []Attr
}

type Name struct {
    Space, Local string
}

type Attr struct {
    Name  Name
    Value string
}
```

`EndElement`

This is also a structure

```go
type EndElement struct {
    Name Name
}
```
    	
`CharData`

This type represents the text content enclosed by a tag and is a simple type

```go
type CharData []byte
```
    	

`Comment`

Similarly for this type

`type Comment []byte`
    	

`ProcInst`

A ProcInst represents an XML processing instruction of the form `<?target inst?>`

```go
type ProcInst struct {
    Target string
    Inst   []byte
}
```
	
`Directive`

A Directive represents an XML directive of the form <!text>. The bytes do not include the <! and > markers.

```go
type Directive []byte
```
    	
A program to print out the tree structure of an XML document is

```go
/* Parse XML
 */

package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
	"os"
	"strings"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "file")
		os.Exit(1)
	}
	file := os.Args[1]
	bytes, err := ioutil.ReadFile(file)
	checkError(err)
	r := strings.NewReader(string(bytes))

	parser := xml.NewDecoder(r)
	depth := 0
	for {
		token, err := parser.Token()
		if err != nil {
			break
		}
		switch t := token.(type) {
		case xml.StartElement:
			elmt := xml.StartElement(t)
			name := elmt.Name.Local
			printElmt(name, depth)
			depth++
		case xml.EndElement:
			depth--
			elmt := xml.EndElement(t)
			name := elmt.Name.Local
			printElmt(name, depth)
		case xml.CharData:
			bytes := xml.CharData(t)
			printElmt("\""+string([]byte(bytes))+"\"", depth)
		case xml.Comment:
			printElmt("Comment", depth)
		case xml.ProcInst:
			printElmt("ProcInst", depth)
		case xml.Directive:
			printElmt("Directive", depth)
		default:
			fmt.Println("Unknown")
		}
	}
}

func printElmt(s string, depth int) {
	for n := 0; n < depth; n++ {
		fmt.Print("  ")
	}
	fmt.Println(s)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

*Note that the parser includes all CharData, including the whitespace between tags.*

If we run this program against the `person` data structure given earlier, it produces

```
person
  "
  "
  name
    "
    "
    family
      " Newmarch "
    family
    "
    "
    personal
      " Jan "
    personal
    "
  "
  name
  "
  "
  email
    "
    jan@newmarch.name
  "
  email
  "
  "
  email
    "
    j.newmarch@boxhill.edu.au
  "
  email
  "
"
person
"
"
```

Note that as no DTD or other XML specification has been used, the tokenizer correctly prints out all the white space (a DTD may specify that the whitespace can be ignored, but without it that assumption cannot be made.)

There is a potential trap in using this parser. It re-uses space for strings, so that once you see a token you need to copy its value if you want to refer to it later. Go has methods such as 
`func (c CharData) Copy() CharData` to make a copy of data. 

