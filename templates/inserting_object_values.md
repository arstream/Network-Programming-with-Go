## Inserting object values

A template is applied to a Go object. Fields from that Go object can be inserted into the template, and you can 'dig' into the object to find subfields, etc. The current object is represented as `'.'`, so that to insert the value of the current object as a string, you use `{{.}}`. The package uses the `fmt` package by default to work out the string used as inserted values.

To insert the value of a field of the current object, you use the field name prefixed by `'.'`. For example, if the object is of type

```go
type Person struct {
        Name      string
        Age       int
        Emails     []string
        Jobs       []*Jobs
}
```


then you insert the values of `Name` and `Age` by

```
The name is {{.Name}}.
The age is {{.Age}}.
```

We can loop over the elements of an array or other list using the `range` command. So to access the contents of the `Emails` array we do

```
{{range .Emails}}
        ...
{{end}}
```

if `Job` is defined by

```go
type Job struct {
    Employer string
    Role     string
}
```

and we want to access the fields of a `Person`'s `Jobs`, we can do it as above with a `{{range .Jobs}}`. An alternative is to switch the current object to the `Jobs` field. This is done using the `{{with ...}} ... {{end}}` construction, where now `{{.}}` is the `Jobs` field, which is an array:

```
{{with .Jobs}}
    {{range .}}
        An employer is {{.Employer}}
        and the role is {{.Role}}
    {{end}}
{{end}}
```

You can use this with any field, not just an array. Using templates

Once we have a template, we can apply it to an object to generate a new string, using the object to fill in the template values. This is a two-step process which involves parsing the template and then applying it to an object. The result is output to a `Writer`, as in

```go
t := template.New("Person template")
t, err := t.Parse(templ)
if err == nil {
	buff := bytes.NewBufferString("")
	t.Execute(buff, person)
}
```

An example program to apply a template to an object and print to standard output is

```go
/**
 * PrintPerson
 */

package main

import (
	"fmt"
	"html/template"
	"os"
)

type Person struct {
	Name   string
	Age    int
	Emails []string
	Jobs   []*Job
}

type Job struct {
	Employer string
	Role     string
}

const templ = `The name is {{.Name}}.
The age is {{.Age}}.
{{range .Emails}}
        An email is {{.}}
{{end}}

{{with .Jobs}}
    {{range .}}
        An employer is {{.Employer}}
        and the role is {{.Role}}
    {{end}}
{{end}}
`

func main() {
	job1 := Job{Employer: "Monash", Role: "Honorary"}
	job2 := Job{Employer: "Box Hill", Role: "Head of HE"}

	person := Person{
		Name:   "jan",
		Age:    50,
		Emails: []string{"jan@newmarch.name", "jan.newmarch@gmail.com"},
		Jobs:   []*Job{&job1, &job2},
	}

	t := template.New("Person template")
	t, err := t.Parse(templ)
	checkError(err)

	err = t.Execute(os.Stdout, person)
	checkError(err)
}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

The output from this is

```
The name is jan.
The age is 50.

An email is jan@newmarch.name

An email is jan.newmarch@gmail.com

An employer is Monash
and the role is Honorary

An employer is Box Hill
and the role is Head of HE
```  


Note that there is plenty of whitespace as newlines in this printout. This is due to the whitespace we have in our template. If we wish to reduce this, eliminate newlines in the template as in

```
{{range .Emails}} An email is {{.}} {{end}}
```


In the example, we used a string in the program as the template. You can also load templates from a file using the function `template.ParseFiles()`. For some reason that I don't understand (and which wasn't required in earlier versions), the name assigned to the template must be the same as the basename of the first file in the list of files. Is this a bug? 

