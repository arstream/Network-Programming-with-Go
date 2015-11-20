## The Complete Server

 The complete server is

```go
/* Server
 */

package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
	"regexp"
	"text/template"
)

import (
	"dictionary"
	"flashcards"
	"templatefuncs"
)

var d *dictionary.Dictionary

func main() {
	if len(os.Args) != 2 {
		fmt.Fprint(os.Stderr, "Usage: ", os.Args[0], ":port\n")
		os.Exit(1)
	}
	port := os.Args[1]

	// dictionaryPath := "/var/www/go/chinese/cedict_ts.u8"
	dictionaryPath := "cedict_ts.u8"
	d = new(dictionary.Dictionary)
	d.Load(dictionaryPath)
	fmt.Println("Loaded dict", len(d.Entries))

	http.HandleFunc("/", listFlashCards)
	//fileServer := http.FileServer("/var/www/go/chinese/jscript", "/jscript/")	
	fileServer := http.StripPrefix("/jscript/", http.FileServer(http.Dir("jscript")))
	http.Handle("/jscript/", fileServer)
	// fileServer = http.FileServer("/var/www/go/chinese/html", "/html/")
	fileServer = http.StripPrefix("/html/", http.FileServer(http.Dir("html")))
	http.Handle("/html/", fileServer)

	http.HandleFunc("/wordlook", lookupWord)
	http.HandleFunc("/flashcards.html", listFlashCards)
	http.HandleFunc("/flashcardSets", manageFlashCards)
	http.HandleFunc("/searchWord", searchWord)
	http.HandleFunc("/addWord", addWord)
	http.HandleFunc("/newFlashCardSet", newFlashCardSet)

	// deliver requests to the handlers
	err := http.ListenAndServe(port, nil)
	checkError(err)
	// That's it!
}

func indexPage(rw http.ResponseWriter, req *http.Request) {
	index, _ := ioutil.ReadFile("html/index.html")
	rw.Write([]byte(index))
}

func lookupWord(rw http.ResponseWriter, req *http.Request) {
	word := req.FormValue("word")
	words := d.LookupEnglish(word)

	//t := template.New("PinyinTemplate")
	t := template.New("DictionaryEntry.html")
	t = t.Funcs(template.FuncMap{"pinyin": templatefuncs.PinyinFormatter})
	t, err := t.ParseFiles("html/DictionaryEntry.html")
	if err != nil {
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	t.Execute(rw, words)
}

type DictPlus struct {
	*dictionary.Dictionary
	Word     string
	CardName string
}

func searchWord(rw http.ResponseWriter, req *http.Request) {
	word := req.FormValue("word")
	searchType := req.FormValue("searchtype")
	cardName := req.FormValue("cardname")

	var words *dictionary.Dictionary
	var dp []DictPlus
	if searchType == "english" {
		words = d.LookupEnglish(word)
		d1 := DictPlus{Dictionary: words, Word: word, CardName: cardName}
		dp = make([]DictPlus, 1)
		dp[0] = d1
	} else {
		words = d.LookupPinyin(word)
		numTrans := 0
		for _, entry := range words.Entries {
			numTrans += len(entry.Translations)
		}
		dp = make([]DictPlus, numTrans)
		idx := 0
		for _, entry := range words.Entries {
			for _, trans := range entry.Translations {
				dict := new(dictionary.Dictionary)
				dict.Entries = make([]*dictionary.Entry, 1)
				dict.Entries[0] = entry
				dp[idx] = DictPlus{
					Dictionary: dict,
					Word:       trans,
					CardName:   cardName}
				idx++
			}
		}
	}

	//t := template.New("PinyinTemplate")
	t := template.New("ChooseDictionaryEntry.html")
	t = t.Funcs(template.FuncMap{"pinyin": templatefuncs.PinyinFormatter})
	t, err := t.ParseFiles("html/ChooseDictionaryEntry.html")
	if err != nil {
		fmt.Println(err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	t.Execute(rw, dp)
}

func newFlashCardSet(rw http.ResponseWriter, req *http.Request) {
	defer http.Redirect(rw, req, "http:/flashcards.html", 200)

	newSet := req.FormValue("NewFlashcard")
	fmt.Println("New cards", newSet)
	// check against nasties:
	b, err := regexp.Match("[/$~]", []byte(newSet))
	if err != nil {
		return
	}
	if b {
		fmt.Println("No good string")
		return
	}

	flashcards.NewFlashCardSet(newSet)
	return
}

func addWord(rw http.ResponseWriter, req *http.Request) {
	url := req.URL
	fmt.Println("url", url.String())
	fmt.Println("query", url.RawQuery)

	word := req.FormValue("word")
	cardName := req.FormValue("cardname")
	simplified := req.FormValue("simplified")
	pinyin := req.FormValue("pinyin")
	traditional := req.FormValue("traditional")
	translations := req.FormValue("translations")

	fmt.Println("word is ", word, " card is ", cardName,
		" simplified is ", simplified, " pinyin is ", pinyin,
		" trad is ", traditional, " trans is ", translations)
	flashcards.AddFlashEntry(cardName, word, pinyin, simplified,
		traditional, translations)
	// add another card?
	addFlashCards(rw, cardName)
}

func listFlashCards(rw http.ResponseWriter, req *http.Request) {

	flashCardsNames := flashcards.ListFlashCardsNames()
	t, err := template.ParseFiles("html/ListFlashcards.html")
	if err != nil {
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	t.Execute(rw, flashCardsNames)
}

/* 
 * Called from ListFlashcards.html on form submission
 */
func manageFlashCards(rw http.ResponseWriter, req *http.Request) {

	set := req.FormValue("flashcardSets")
	order := req.FormValue("order")
	action := req.FormValue("submit")
	half := req.FormValue("half")
	fmt.Println("set chosen is", set)
	fmt.Println("order is", order)
	fmt.Println("action is", action)

	cardname := "flashcardSets/" + set

	//components := strings.Split(req.URL.Path[1:], "/", -1)
	//cardname := components[1]
	//action := components[2]
	fmt.Println("cardname", cardname, "action", action)
	if action == "Show cards in set" {
		showFlashCards(rw, cardname, order, half)
	} else if action == "List words in set" {
		listWords(rw, cardname)
	} else if action == "Add cards to set" {
		addFlashCards(rw, set)
	}
}

func showFlashCards(rw http.ResponseWriter, cardname, order, half string) {
	fmt.Println("Loading card name", cardname)
	cards := new(flashcards.FlashCards)
	//cards.Load(cardname, d)
	//flashcards.SaveJSON(cardname + ".json", cards)
	flashcards.LoadJSON(cardname, &cards)
	if order == "Sequential" {
		cards.CardOrder = "SEQUENTIAL"
	} else {
		cards.CardOrder = "RANDOM"
	}
	fmt.Println("half is", half)
	if half == "Random" {
		cards.ShowHalf = "RANDOM_HALF"
	} else if half == "English" {
		cards.ShowHalf = "ENGLISH_HALF"
	} else {
		cards.ShowHalf = "CHINESE_HALF"
	}
	fmt.Println("loaded cards", len(cards.Cards))
	fmt.Println("Card name", cards.Name)

	//t := template.New("PinyinTemplate")
	t := template.New("ShowFlashcards.html")
	t = t.Funcs(template.FuncMap{"pinyin": templatefuncs.PinyinFormatter})
	t, err := t.ParseFiles("html/ShowFlashcards.html")
	if err != nil {
		fmt.Println(err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	err = t.Execute(rw, cards)
	if err != nil {
		fmt.Println("Execute error " + err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
}

func listWords(rw http.ResponseWriter, cardname string) {
	fmt.Println("Loading card name", cardname)
	cards := new(flashcards.FlashCards)
	//cards.Load(cardname, d)
	flashcards.LoadJSON(cardname, cards)
	fmt.Println("loaded cards", len(cards.Cards))
	fmt.Println("Card name", cards.Name)

	//t := template.New("PinyinTemplate")
	t := template.New("ListWords.html")
	if t.Tree == nil || t.Root == nil {
		fmt.Println("New t is an incomplete or empty template")
	}
	t = t.Funcs(template.FuncMap{"pinyin": templatefuncs.PinyinFormatter})
	t, err := t.ParseFiles("html/ListWords.html")
	if t.Tree == nil || t.Root == nil {
		fmt.Println("Parsed t is an incomplete or empty template")
	}

	if err != nil {
		fmt.Println("Parse error " + err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	err = t.Execute(rw, cards)
	if err != nil {
		fmt.Println("Execute error " + err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	fmt.Println("No error ")
}

func addFlashCards(rw http.ResponseWriter, cardname string) {
	t, err := template.ParseFiles("html/AddWordToSet.html")
	if err != nil {
		fmt.Println("Parse error " + err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}
	cards := flashcards.GetFlashCardsByName(cardname, d)
	t.Execute(rw, cards)
	if err != nil {
		fmt.Println("Execute error " + err.Error())
		http.Error(rw, err.Error(), http.StatusInternalServerError)
		return
	}

}

func checkError(err error) {
	if err != nil {
		fmt.Println("Fatal error ", err.Error())
		os.Exit(1)
	}
}
```

