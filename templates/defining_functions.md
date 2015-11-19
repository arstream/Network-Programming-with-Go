## Defining functions

The templates use the string representation of an object to insert values, using the `fmt` package to convert the object to a string. Sometimes this isn't what is needed. For example, to avoid spammers getting hold of email addresses it is quite common to see the symbol `'@'` replaced by the word `" at "`, as in `"jan at newmarch.name"`. If we want to use a template to display email addresses in that form, then we have to build a custom function to do this transformation.

Each template function has a name that is used in the templates themselves, and an associated Go function. These are linked by the type

```go
type FuncMap map[string]interface{}
```

For example, if we want our template function to be `"emailExpand"` which is linked to the Go function `EmailExpander` then we add this to the functions in a template by

```go
t = t.Funcs(template.FuncMap{"emailExpand": EmailExpander})
```

The signature for `EmailExpander` is typically

```go
func EmailExpander(args ...interface{}) string
```

In the use we are interested in, there should only be one argument to the function which will be a string. Existing functions in the Go template library have some initial code to handle non-conforming cases, so we just copy that. Then it is just simple string manipulation to change the format of the email address. A program is

```go
/**
 * PrintEmails
 */

package main

import (
	"fmt"
	"os"
	"strings"
	"text/template"
)

type Person struct {
	Name   string
	Emails []string
}

const templ = `The name is {{.Name}}.
{{range .Emails}}
        An email is "{{. | emailExpand}}"
{{end}}
`

func EmailExpander(args ...interface{}) string {
	ok := false
	var s string
	if len(args) == 1 {
		s, ok = args[0].(string)
	}
	if !ok {
		s = fmt.Sprint(args...)
	}

	// find the @ symbol
	substrs := strings.Split(s, "@")
	if len(substrs) != 2 {
		return s
	}
	// replace the @ by " at "
	return (substrs[0] + " at " + substrs[1])
}

func main() {
	person := Person{
		Name:   "jan",
		Emails: []string{"jan@newmarch.name", "jan.newmarch@gmail.com"},
	}

	t := template.New("Person template")

	// add our function
	t = t.Funcs(template.FuncMap{"emailExpand": EmailExpander})

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

The output is

```
The name is jan.

    An email is "jan at newmarch.name"
    
    An email is "jan.newmarch at gmail.com"
```
