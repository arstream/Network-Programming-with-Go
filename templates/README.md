# Chapter 9 Templates

Many languages have mechanisms to convert strings from one form to another. Go has a template mechanism to convert strings based on the content of an object supplied as an argument. While this is often used in rewriting HTML to insert object values, it can be used in other situations. Note that this material doesn't have anything explicitly to do with networking, but may be useful to network programs. 

## Introduction

Most server-side languages have a mechanism for taking predominantly static pages and inserting a dynamically generated component, such as a list of items. Typical examples are scripts in Java Server Pages, PHP scripting and many others. Go has adopted a relatively simple scripting language in the `template` package.

We describe the new package here. The package is designed to take text as input and output different text, based on transforming the original text using the values of an object. Unlike JSP or similar, it is not restricted to HTML files but it is likely to find greatest use there.

The original source is called a *template* and will consist of text that is transmitted unchanged, and embedded commands which can act on and change text. The commands are delimited by `{{ ... }}` , similar to the JSP commands `<%= ... =%>` and PHPs `<?php ... ?>`. 