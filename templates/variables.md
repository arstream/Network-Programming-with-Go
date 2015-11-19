## Variables

The template package allows you to define and use variables. As motivation for this, consider how we might print each person's email address *prefixed* by their name. The type we use is again

```go
type Person struct {
        Name      string
        Emails     []string
}
```

To access the email strings, we use a `range` statement such as

```
{{range .Emails}}
    {{.}}
{{end}}
```

But at that point we cannot access the `Name` field as `'.'` is now traversing the array elements and the `Name` is outside of this scope. The solution is to save the value of the `Name` field in a variable that can be accessed anywhere in its scope. Variables in templates are prefixed by `'$'`. So we write

```
{{$name := .Name}}
{{range .Emails}}
    Name is {{$name}}, email is {{.}}
{{end}}
```

The program is

```go
/**
 * PrintNameEmails
 */

package main

import (
	"html/template"
	"os"
	"fmt"
)

type Person struct {
	Name   string
	Emails []string
}

const templ = `{{$name := .Name}}
{{range .Emails}}
    Name is {{$name}}, email is {{.}}
{{end}}
`

func main() {
	person := Person{
		Name:   "jan",
		Emails: []string{"jan@newmarch.name", "jan.newmarch@gmail.com"},
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

with output

```
Name is jan, email is jan@newmarch.name

Name is jan, email is jan.newmarch@gmail.com
```


