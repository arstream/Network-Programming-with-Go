## The Chinese Dictionary

Chinese is a complex language (aren't they all :-( ). The written form is hieroglyphic, that is "pictograms" instead of using an alphabet. But this written form has evolved over time, and even recently split into two forms: "traditional" Chinese as used in Taiwan and Hong Kong, and "simplified" Chinese as used in mainland China. While most of the characters are the same, about 1,000 are different. Thus a Chinese dictionary will often have two written forms of the same character.

Most Westerners like me can't understand these characters. So there is a "Latinised" form called Pinyin which writes the characters in a phonetic alphabet based on the Latin alphabet. It isn't quite the Latin alphabet, because Chinese is a tonal language, and the Pinyin form has to show the tones (much like acccents in French and other European languages). So a typical dictionary has to show four things: the traditional form, the simplified form, the Pinyin and the English. For example,

Traditional | Simplified| Pinyin | English
-|-|-|-
好 | 好 | hǎo | good

But again there is a little complication. There is a free [Chinese/English dictionary](http://www.mandarintools.com/worddict.html) and even better, you can download it as a UTF-8 file, which Go is well suited to handle. In this, the Chinese characters are written in Unicode but the Pinyin characters are not: although there are Unicode characters for letters such as 'ǎ', many dictionaries including this one use the Latin 'a' and place the tone at the end of the word. Here it is the third tone, so "hǎo" is written as "hao3". This makes it easier for those who only have US keyboards and no Unicode editor to still communicate in Pinyin.

This data format mismatch is not a big deal: just that somewhere along the line, between the original text dictionary and the display in the browser, a data massage has to be performed. Go templates allow this to be done by defining a custom template, so I chose that route. Alternatives could have been to do this as the dictionary is read in, or in the Javascript to display the final characters.

The code for the Pinyin formatter is given below. Please don't bother reading it unless you are *really* interested in knowing the rules for Pinyin formatting. 

```go
package pinyin

import (
	"io"
	"strings"
)

func PinyinFormatter(w io.Writer, format string, value ...interface{}) {
	line := value[0].(string)
	words := strings.Fields(line)
	for n, word := range words {
		// convert "u:" to "ü" if present
		uColon := strings.Index(word, "u:")
		if uColon != -1 {
			parts := strings.SplitN(word, "u:", 2)
			word = parts[0] + "ü" + parts[1]
		}
		println(word)
		// get last character, will be the tone if present
		chars := []rune(word)
		tone := chars[len(chars)-1]
		if tone == '5' {
			words[n] = string(chars[0 : len(chars)-1])
			println("lost accent on", words[n])
			continue
		}
		if tone < '1' || tone > '4' {
			continue
		}
		words[n] = addAccent(word, int(tone))
	}
	line = strings.Join(words, ` `)
	w.Write([]byte(line))
}

var (
	// maps 'a1' to '\u0101' etc
	aAccent = map[int]rune{
		'1': '\u0101',
		'2': '\u00e1',
		'3': '\u01ce', // '\u0103',
		'4': '\u00e0'}
	eAccent = map[int]rune{
		'1': '\u0113',
		'2': '\u00e9',
		'3': '\u011b', // '\u0115',
		'4': '\u00e8'}
	iAccent = map[int]rune{
		'1': '\u012b',
		'2': '\u00ed',
		'3': '\u01d0', // '\u012d',
		'4': '\u00ec'}
	oAccent = map[int]rune{
		'1': '\u014d',
		'2': '\u00f3',
		'3': '\u01d2', // '\u014f',
		'4': '\u00f2'}
	uAccent = map[int]rune{
		'1': '\u016b',
		'2': '\u00fa',
		'3': '\u01d4', // '\u016d',
		'4': '\u00f9'}
	üAccent = map[int]rune{
		'1': 'ǖ',
		'2': 'ǘ',
		'3': 'ǚ',
		'4': 'ǜ'}
)

func addAccent(word string, tone int) string {
	/*
	 * Based on "Where do the tone marks go?"
	 * at http://www.pinyin.info/rules/where.html
	 */

	n := strings.Index(word, "a")
	if n != -1 {
		aAcc := aAccent[tone]
		// replace 'a' with its tone version
		word = word[0:n] + string(aAcc) + word[(n+1):len(word)-1]
	} else {
		n := strings.Index(word, "e")
		if n != -1 {
			eAcc := eAccent[tone]
			word = word[0:n] + string(eAcc) +
				word[(n+1):len(word)-1]
		} else {
			n = strings.Index(word, "ou")
			if n != -1 {
				oAcc := oAccent[tone]
				word = word[0:n] + string(oAcc) + "u" +
					word[(n+2):len(word)-1]
			} else {
				chars := []rune(word)
				length := len(chars)
				// put tone onthe last vowel
			L:
				for n, _ := range chars {
					m := length - n - 1
					switch chars[m] {
					case 'i':
						chars[m] = iAccent[tone]
						break L
					case 'o':
						chars[m] = oAccent[tone]
						break L
					case 'u':
						chars[m] = uAccent[tone]
						break L
					case 'ü':
						chars[m] = üAccent[tone]
						break L
					default:
					}
				}
				word = string(chars[0 : len(chars)-1])
			}
		}
	}

	return word
}
```

How this is used is illustrated by the function `lookupWord`. This is called in response to an HTML Form request to find the English words in a dictionary.

```go
func lookupWord(rw http.ResponseWriter, req *http.Request) {
        word := req.FormValue("word")
        words := d.LookupEnglish(word)

        pinyinMap := template.FormatterMap {"pinyin": pinyin.PinyinFormatter}
        t, err := template.ParseFile("html/DictionaryEntry.html", pinyinMap)
        if err != nil {
                http.Error(rw, err.String(), http.StatusInternalServerError)
                return
        }
        t.Execute(rw, words)
}
```
  

The HTML code is

```html
<html>
  <body>
    <table border="1">
      <tr>
	<th>Word</th>
	<th>Traditional</th>
	<th>Simplified</th>
	<th>Pinyin</th>
	<th>English</th>
      </tr>
      {{with .Entries}}
      {{range .}}
      {.repeated section Entries}
      <tr>
	<td>{{.Word}}</td>
	<td>{{.Traditional}}</td> 
	<td>{{.Simplified}}</td>
	<td>{{.Pinyin|pinyin}}</td>
	<td>
	  <pre>
	    {.repeated section Translations} 
	    {@|html} 
	    {.end}
	  </pre>
	</td>
      </tr>
      {.end} 
      {{end}}
      {{end}}
    </table>
  </body>
</html>
```

### The Dictionary type

The text file containing the dictionary has lines of the form <br>
*traditional simplified [pinyin] /translation/translation/.../*

For example,<br>
好 好 [hao3] /good/well/proper/good to/easy to/very/so/(suffix indicating completion or readiness)/

We store each line as an `Entry` within the `Dictionary` package:

```go
type Entry struct {
    Traditional string
    Simplified string
    Pinyin     string
    Translations []string
}
```

The dictionary itself is just an array of these entries:

```go 
type Dictionary struct {
    Entries []*Entry
}
```

Building the dictionary is easy enough. Just read each line and break the line into its various bits using simple string methods. Then add the line to the dictionary slice.

Looking up entries in this dictionary is straightforward: just search through until we find the appropriate key. There are about 100,000 entries in this dictionary: brute force by a linear search is fast enough. If it were necessary, faster storage and search mechanisms could easily be used.

The original dictionary grows by people on the Web adding in entries as they see fit. Consequently it isn't that well organised and contains repetitions and multiple entries. So looking up any word - either by Pinyin or by English - may return multiple matches. To cater for this, each lookup returns a "mini dictionary", just those lines in the full dictionary that match.

The Dictionary code is

```go
package dictionary

import (
	"bufio"
	//"fmt"
	"os"
	"strings"
)

type Entry struct {
	Traditional  string
	Simplified   string
	Pinyin       string
	Translations []string
}

func (de Entry) String() string {
	str := de.Traditional + ` ` + de.Simplified + ` ` + de.Pinyin
	for _, t := range de.Translations {
		str = str + "\n    " + t
	}
	return str
}

type Dictionary struct {
	Entries []*Entry
}

func (d *Dictionary) String() string {
	str := ""
	for n := 0; n < len(d.Entries); n++ {
		de := d.Entries[n]
		str += de.String() + "\n"
	}
	return str
}

func (d *Dictionary) LookupPinyin(py string) *Dictionary {
	newD := new(Dictionary)
	v := make([]*Entry, 0, 100)
	for n := 0; n < len(d.Entries); n++ {
		de := d.Entries[n]
		if de.Pinyin == py {
			v = append(v, de)
		}
	}
	newD.Entries = v
	return newD
}

func (d *Dictionary) LookupEnglish(eng string) *Dictionary {
	newD := new(Dictionary)
	v := make([]*Entry, 0, 100)
	for n := 0; n < len(d.Entries); n++ {
		de := d.Entries[n]
		for _, e := range de.Translations {
			if e == eng {
				v = append(v, de)
			}
		}
	}
	newD.Entries = v
	return newD
}

func (d *Dictionary) LookupSimplified(simp string) *Dictionary {
	newD := new(Dictionary)
	v := make([]*Entry, 0, 100)

	for n := 0; n < len(d.Entries); n++ {
		de := d.Entries[n]
		if de.Simplified == simp {
			v = append(v, de)
		}
	}
	newD.Entries = v
	return newD
}

func (d *Dictionary) Load(path string) {

	f, err := os.Open(path)
	r := bufio.NewReader(f)
	if err != nil {
		println(err.Error())
		os.Exit(1)
	}

	v := make([]*Entry, 0, 100000)
	numEntries := 0
	for {
		line, err := r.ReadString('\n')
		if err != nil {
			break
		}
		if line[0] == '#' {
			continue
		}
		// fmt.Println(line)
		trad, simp, pinyin, translations := parseDictEntry(line)

		de := Entry{
			Traditional:  trad,
			Simplified:   simp,
			Pinyin:       pinyin,
			Translations: translations}

		v = append(v, &de)
		numEntries++
	}
	// fmt.Printf("Num entries %d\n", numEntries)
	d.Entries = v
}

func parseDictEntry(line string) (string, string, string, []string) {
	// format is
	//    trad simp [pinyin] /trans/trans/.../
	tradEnd := strings.Index(line, " ")
	trad := line[0:tradEnd]
	line = strings.TrimSpace(line[tradEnd:])

	simpEnd := strings.Index(line, " ")
	simp := line[0:simpEnd]
	line = strings.TrimSpace(line[simpEnd:])

	pinyinEnd := strings.Index(line, "]")
	pinyin := line[1:pinyinEnd]
	line = strings.TrimSpace(line[pinyinEnd+1:])

	translations := strings.Split(line, "/")
	// includes empty at start and end, so
	translations = translations[1 : len(translations)-1]

	return trad, simp, pinyin, translations
}
```


