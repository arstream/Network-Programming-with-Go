# Chapter 12 XML

XML is a significant markup language mainly intended as a means of serialising data structures as a text document. Go has basic support for XML document processing.

## Introduction

 XML is now a widespread way of representing complex data structures serialised into text format. It is used to describe documents such as DocBook and XHTML. It is used in specialised markup languages such as MathML and CML (Chemistry Markup Language). It is used to encode data as SOAP messages for Web Services, and the Web Service can be specified using WSDL (Web Services Description Language).

At the simplest level, XML allows you to define your own tags for use in text documents. Tags can be nested and can be interspersed with text. Each tag can also contain attributes with values. For example,

```html
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
    
The structure of any XML document can be described in a number of ways:

* A document type definition DTD is good for describing structure
* XML schema are good for describing the data types used by an XML document
* RELAX NG is proposed as an alternative to both

There is argument over the relative value of each way of defining the structure of an XML document. We won't buy into that, as Go does not suport any of them. Go cannot check for validity of any document against a schema, but only for well-formedness.

Four topics are discussed in this chapter: parsing an XML stream, marshalling and unmarshalling Go data into XML, and XHTML. 