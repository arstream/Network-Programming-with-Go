## XHTML and HTML

### XHTML

HTML does not conform to XML syntax. It has unterminated tags such as `<br>`. XHTML is a cleanup of HTML to make it compliant to XML. Documents in XHTML can be managed using the techniques above for XML. 

### HTML

There is some support in the XML package to handle HTML documents even though they are not XML-compliant. The XML parser discussed earlier can handle many HTML documents if it is modified by

```go 
parser := xml.NewDecoder(r)
parser.Strict = false
parser.AutoClose = xml.HTMLAutoClose
parser.Entity = xml.HTMLEntity
```

## Conclusion

Go has basic support for dealing with XML strings. It does not as yet have mechanisms for dealing with XML specification languages such as XML Schema or Relax NG.

  



