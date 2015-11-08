# JSON

JSON stands for JavaScript Object Notation. It was designed to be a lighweight means of passing data between JavaScript systems. It uses a text-based format and is sufficiently general that it has become used as a general purpose serialisation method for many programming languages.

JSON serialises objects, arrays and basic values. The basic values include string, number, boolean values and the null value. Arrays are a comma-separated list of values that can represent arrays, vectors, lists or sequences of various programming languages. They are delimited by square brackets `[ ... ]`. Objects are represented by a list of "field: value" pairs enclosed in curly braces `{ ... }`.

For example, the table of employees given earlier could be written as an array of employee objects:

```
[
   {Name: fred, Occupation: programmer},
   {Name: liping, Occupation: analyst},
   {Name: sureerat, Occupation: manager}
]
```

There is no special support for complex data types such as dates, no distinction between number types, no recursive types, etc. JSON is a very simple language, but nevertheless can be quite useful. Its text-based format makes it easy for people to use, even though it has the overheads of string handling.

From the Go JSON package specification, marshalling uses the following type-dependent default encodings:

* Boolean values encode as JSON booleans.
* Floating point and integer values encode as JSON numbers.
* String values encode as JSON strings, with each invalid UTF-8 sequence replaced by the encoding of the Unicode replacement character U+FFFD.
* Array and slice values encode as JSON arrays, except that []byte encodes as a base64-encoded string.
* Struct values encode as JSON objects. Each struct field becomes a member of the object. By default the object's key name is the struct field name converted to lower case. If the struct field has a tag, that tag will be used as the name instead.
* Map values encode as JSON objects. The map's key type must be string; the object keys are used directly as map keys.
* Pointer values encode as the value pointed to. (Note: this allows trees, but not graphs!). A nil pointer  encodes as the null JSON object.
* Interface values encode as the value contained in the interface. A nil interface value encodes as the null  JSON object.
* Channel, complex, and function values cannot be encoded in JSON. Attempting to encode such a value cause Marshal to return an `InvalidTypeError`.
* JSON cannot represent cyclic data structures and Marshal does not handle them. Passing cyclic structures to Marshal will result in an infinite recursion.


A program to store JSON serialised data into a file is

```go
/* SaveJSON
 */

package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Person struct {
	Name  Name
	Email []Email
}

type Name struct {
	Family   string
	Personal string
}

type Email struct {
	Kind    string
	Address string
}

func main() {
	person := Person{
		Name: Name{Family: "Newmarch", Personal: "Jan"},
		Email: []Email{Email{Kind: "home", Address: "jan@newmarch.name"},
			Email{Kind: "work", Address: "j.newmarch@boxhill.edu.au"}}}

	saveJSON("person.json", person)
}

func saveJSON(fileName string, key interface{}) {
	outFile, err := os.Create(fileName)
	checkError(err)
	encoder := json.NewEncoder(outFile)
	err = encoder.Encode(key)
	checkError(err)
	outFile.Close()
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

and to load it back into memory is

```go
/* LoadJSON
 */

package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Person struct {
	Name  Name
	Email []Email
}

type Name struct {
	Family   string
	Personal string
}

type Email struct {
	Kind    string
	Address string
}

func (p Person) String() string {
	s := p.Name.Personal + " " + p.Name.Family
	for _, v := range p.Email {
		s += "\n" + v.Kind + ": " + v.Address
	}
	return s
}
func main() {
	var person Person
	loadJSON("person.json", &person)

	fmt.Println("Person", person.String())
}

func loadJSON(fileName string, key interface{}) {
	inFile, err := os.Open(fileName)
	checkError(err)
	decoder := json.NewDecoder(inFile)
	err = decoder.Decode(key)
	checkError(err)
	inFile.Close()
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

The serialised form is (formatted nicely)

```
{"Name":{"Family":"Newmarch",
         "Personal":"Jan"},
 "Email":[{"Kind":"home","Address":"jan@newmarch.name"},
          {"Kind":"work","Address":"j.newmarch@boxhill.edu.au"}
         ]
}
```

### A client and server

A client to send a person's data and read it back ten times is

```go
/* JSON EchoClient
 */
package main

import (
	"fmt"
	"net"
	"os"
	"encoding/json"
	"bytes"
	"io"
)

type Person struct {
	Name  Name
	Email []Email
}

type Name struct {
	Family   string
	Personal string
}

type Email struct {
	Kind    string
	Address string
}

func (p Person) String() string {
	s := p.Name.Personal + " " + p.Name.Family
	for _, v := range p.Email {
		s += "\n" + v.Kind + ": " + v.Address
	}
	return s
}

func main() {
	person := Person{
		Name: Name{Family: "Newmarch", Personal: "Jan"},
		Email: []Email{Email{Kind: "home", Address: "jan@newmarch.name"},
			Email{Kind: "work", Address: "j.newmarch@boxhill.edu.au"}}}

	if len(os.Args) != 2 {
		fmt.Println("Usage: ", os.Args[0], "host:port")
		os.Exit(1)
	}
	service := os.Args[1]

	conn, err := net.Dial("tcp", service)
	checkError(err)

	encoder := json.NewEncoder(conn)
	decoder := json.NewDecoder(conn)

	for n := 0; n < 10; n++ {
		encoder.Encode(person)
		var newPerson Person
		decoder.Decode(&newPerson)
		fmt.Println(newPerson.String())
	}

	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
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

and the corresponding server is

```go
/* JSON EchoServer
 */
package main

import (
	"fmt"
	"net"
	"os"
	"encoding/json"
)

type Person struct {
	Name  Name
	Email []Email
}

type Name struct {
	Family   string
	Personal string
}

type Email struct {
	Kind    string
	Address string
}

func (p Person) String() string {
	s := p.Name.Personal + " " + p.Name.Family
	for _, v := range p.Email {
		s += "\n" + v.Kind + ": " + v.Address
	}
	return s
}

func main() {

	service := "0.0.0.0:1200"
	tcpAddr, err := net.ResolveTCPAddr("tcp", service)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}

		encoder := json.NewEncoder(conn)
		decoder := json.NewDecoder(conn)

		for n := 0; n < 10; n++ {
			var person Person
			decoder.Decode(&person)
			fmt.Println(person.String())
			encoder.Encode(person)
		}
		conn.Close() // we're finished
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```