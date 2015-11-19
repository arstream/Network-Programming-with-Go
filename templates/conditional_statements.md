## Conditional statements

Continuing with our `Person` example, supposing we just want to print out the list of emails, without digging into it. We can do that with a template

```
Name is {{.Name}}
Emails are {{.Emails}}
```

This will print

```
Name is jan
Emails are [jan@newmarch.name jan.newmarch@gmail.com]
```

because that is how the `fmt` package will display a list.

In many circumstances that may be fine, if that is what you want. Let's consider a case where it is *almost* right but not quite. There is a JSON package to serialise objects, which we looked at in Chapter 4. This would produce

```
{"Name": "jan",
 "Emails": ["jan@newmarch.name", "jan.newmarch@gmail.com"]
}
```

The JSON package is the one you would use in practice, but let's see if we can produce JSON output using templates. We can do something similar just by the templates we have. This is *almost* right as a JSON serialiser:

```
{"Name": "{{.Name}}",
 "Emails": {{.Emails}}
}
```

It will produce

```
{"Name": "jan",
 "Emails": [jan@newmarch.name jan.newmarch@gmail.com]
}
```
which has two problems: the addresses aren't in quotes, and the list elements should be `','` separated.

How about this: looking at the array elements, putting them in quotes and adding commas?

```
{"Name": {{.Name}},
  "Emails": [
   {{range .Emails}}
      "{{.}}",
   {{end}}
  ]
}
```

which will produce

```
{"Name": "jan",
 "Emails": ["jan@newmarch.name", "jan.newmarch@gmail.com",]
}
```
(plus some white space.).

Again, almost correct, but if you look carefully, you will see a trailing `','` after the last list element. According to the JSON syntax (see [json.org](www.json.org), this trailing `','` is not allowed. Implementations may vary in how they deal with this.

What we want is "print every element followed by a `','` except for the last one. This is actually a bit hard to do, so a better way is "print every element preceded by a `','` except for the first one. (I got this tip from "brianb" at Stack Overflow.). This is easier, because the first element has index zero and many programming languages, including the Go template language, treat zero as Boolean false.

One form of the conditional statement is `{{if pipeline}} T1 {{else}} T0 {{end}}`. We need the `pipeline` to be the index into the array of emails. Fortunately, a variation on the `range` statement gives us this. There are two forms which introduce variables

```
{{range $elmt := array}}
{{range $index, $elmt := array}}
```

So we set up a loop through the array, and if the index is false (0) we just print the element, otherwise print it preceded by a `','`. The template is

```
{"Name": "{{.Name}}",
 "Emails": [
 {{range $index, $elmt := .Emails}}
    {{if $index}}
        , "{{$elmt}}"
    {{else}}
         "{{$elmt}}"
    {{end}}
 {{end}}
 ]
}
```

and the full program is

```go
/**
 * PrintJSONEmails
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

const templ = `{"Name": "{{.Name}}",
 "Emails": [
{{range $index, $elmt := .Emails}}
    {{if $index}}
        , "{{$elmt}}"
    {{else}}
         "{{$elmt}}"
    {{end}}
{{end}}
 ]
}
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

This gives the correct JSON output.

Before leaving this section, we note that the problem of formatting a list with comma separators can be approached by defining suitable functions in Go that are made available as template functions. To re-use a well known saying, "There's more than one way to do it!". The following program was sent to me by Roger Peppe:

```go
/**
 * Sequence.go
 * Copyright Roger Peppe
 */

package main

import (
	"errors"
	"fmt"
	"os"
	"text/template"
)

var tmpl = `{{$comma := sequence "" ", "}}
{{range $}}{{$comma.Next}}{{.}}{{end}}
{{$comma := sequence "" ", "}}
{{$colour := cycle "black" "white" "red"}}
{{range $}}{{$comma.Next}}{{.}} in {{$colour.Next}}{{end}}
`

var fmap = template.FuncMap{
	"sequence": sequenceFunc,
	"cycle":    cycleFunc,
}

func main() {
	t, err := template.New("").Funcs(fmap).Parse(tmpl)
	if err != nil {
		fmt.Printf("parse error: %v\n", err)
		return
	}
	err = t.Execute(os.Stdout, []string{"a", "b", "c", "d", "e", "f"})
	if err != nil {
		fmt.Printf("exec error: %v\n", err)
	}
}

type generator struct {
	ss []string
	i  int
	f  func(s []string, i int) string
}

func (seq *generator) Next() string {
	s := seq.f(seq.ss, seq.i)
	seq.i++
	return s
}

func sequenceGen(ss []string, i int) string {
	if i >= len(ss) {
		return ss[len(ss)-1]
	}
	return ss[i]
}

func cycleGen(ss []string, i int) string {
	return ss[i%len(ss)]
}

func sequenceFunc(ss ...string) (*generator, error) {
	if len(ss) == 0 {
		return nil, errors.New("sequence must have at least one element")
	}
	return &generator{ss, 0, sequenceGen}, nil
}

func cycleFunc(ss ...string) (*generator, error) {
	if len(ss) == 0 {
		return nil, errors.New("cycle must have at least one element")
	}
	return &generator{ss, 0, cycleGen}, nil
}
```

# Conclusion

The Go template package is useful for certain kinds of text transformations involving inserting values of objects. It does not have the power of, say, regular expressions, but is faster and in many cases will be easier to use than regular expressions 


