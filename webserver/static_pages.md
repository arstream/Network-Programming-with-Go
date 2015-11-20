## Static pages

 Some pages will just have static content. These can be managed by a `fileServer`. For simplicity I put all of the static HTML pages and CSS files in the `html` directory and all of the JavaScript files in the `jscript` directory. These are then delivered by the Go code

```go  
fileServer := http.FileServer("jscript", "/jscript/")
http.Handle("/jscript/", fileServer)

fileServer = http.FileServer("html", "/html/")
http.Handle("/html/", fileServer)
```



