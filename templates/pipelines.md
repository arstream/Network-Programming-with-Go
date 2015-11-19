## Pipelines


The above transformations insert pieces of text into a template. Those pieces of text are essentially arbitrary, whatever the string values of the fields are. If we want them to appear as part of an HTML document (or other specialised form) then we will have to escape particular sequences of characters. For example, to display arbitrary text in an HTML document we have to change `"<"` to `"&lt;"`. The Go templates have a number of builtin functions, and one of these is the function html. These functions act in a similar manner to Unix pipelines, reading from standard input and writing to standard output.

To take the value of the current object `'.'` and apply HTML escapes to it, you write a "pipeline" in the template

```
{{. | html}}
```

and similarly for other functions.

Mike Samuel has pointed out a convenience function currently in the `exp/template/html` package. If all of the entries in a template need to be passed through the html template function, then the Go function `Escape(t *template.Template)` can take a template and add the `html` function to each node in the template that doesn't already have one. This will be useful for templates used for HTML documents and can form a pattern for similar function uses elsewhere. 
