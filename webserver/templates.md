## Templates

 The list of flashcard sets is open ended, depending on the number of files in a directory. These should not be hard-coded into an HTML page, but the content should be generated as needed. This is an obvious candidate for templates.

The list of files in a directory is generated as a list of strings. These can then be displayed in a table using the template

```
<table>
  {{range .}}
  <tr>
    <td>
      {{.}}
    </td>
  </tr>
</table>
```



