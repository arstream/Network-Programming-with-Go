## Unmarshalling XML

Go provides a function `Unmarshal` and a method `func (*Parser) Unmarshal` to unmarshal XML into Go data structures. The unmarshalling is not perfect: Go and XML are different languages.

We consider a simple example before looking at the details. We take the XML document given earlier of

```xml
<person>
  <name>
    <family> Newmarch </family>
    <personal> Jan </personal>
  </name>
  <email type="personal">
    jan@newmarch.name
  </email>
  <email type="work">
    j.newmarch@boxhill.edu.au
  </email>
</person>
```

We would like to map this onto the Go structures

```go
type Person struct {
	Name Name
	Email []Email
}

type Name struct {
	Family string
	Personal string
}

type Email struct {
	Type string
	Address string
}
```

This requires several comments:

1. Unmarshalling uses the Go reflection package. This requires that all fields by public i.e. start with a capital letter. Earlier versions of Go used case-insensitive matching to match fields such as the XML string "name" to the field `Name`. Now, though, *case-sensitive* matching is used. To perform a match, the structure fields must be tagged to show the XML string that will be matched against. This changes `Person` to

```go
type Person struct {
	Name Name `xml:"name"`
	Email []Email `xml:"email"`
}
```
    	
2. While tagging of fields can attach XML strings to fields, it can't do so with the names of the structures. An additional field is required, with field name "XMLName". This only affects the top-level struct, `Person`

```go
type Person struct {
    XMLName Name `xml:"person"`
	Name Name `xml:"name"`
	Email []Email `xml:"email"`
}
```
    	
3. Repeated tags in the map to a slice in Go

4. Attributes within tags will match to fields in a structure only if the Go field has the tag ",attr". This occurs with the field `Type` of `Email`, where matching the attribute "type" of the "email" tag requires `` `xml:"type,attr"` ``

5. If an XML tag has no attributes and only has character data, then it matches a `string` field by the same name (case-sensitive, though). So the tag `` `xml:"family"` `` with character data "Newmarch" maps to the string field `Family`
    
6. But if the tag has attributes, then it must map to a structure. Go assigns the character data to the field with tag  `,chardata`. This occurs with the "email" data and the field `Address` with tag `,chardata`

A program to unmarshal the document above is

```go
/* Unmarshal
 */

package main

import (
	"encoding/xml"
	"fmt"
	"os"
	//"strings"
)

type Person struct {
	XMLName Name    `xml:"person"`
	Name    Name    `xml:"name"`
	Email   []Email `xml:"email"`
}

type Name struct {
	Family   string `xml:"family"`
	Personal string `xml:"personal"`
}

type Email struct {
	Type    string `xml:"type,attr"`
	Address string `xml:",chardata"`
}

func main() {
	str := `<?xml version="1.0" encoding="utf-8"?>
<person>
  <name>
    <family> Newmarch </family>
    <personal> Jan </personal>
  </name>
  <email type="personal">
    jan@newmarch.name
  </email>
  <email type="work">
    j.newmarch@boxhill.edu.au
  </email>
</person>`

	var person Person

	err := xml.Unmarshal([]byte(str), &person)
	checkError(err)

	// now use the person structure e.g.
	fmt.Println("Family name: \"" + person.Name.Family + "\"")
	fmt.Println("Second email address: \"" + person.Email[1].Address + "\"")
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

(Note the spaces are correct.). The strict rules are given in the package specification. 
