## Flash cards

Each individual flash card is of the type `Flashcard`

```go
type FlashCard struct {
    Simplified string
    English    string
    Dictionary *dictionary.Dictionary
}
```

At present we only store the simplified character and the English translation for that character. We also have a `Dictionary` which will contain only one entry for the entry we will have chosen somewhere.

A set of flash cards is defined by the type

```go
type FlashCards struct {
    Name      string
    CardOrder string
    ShowHalf  string
    Cards     []*FlashCard
}
```

where the `CardOrder` will be `"random"` or `"sequential"` and the `ShowHalf` will be `"RANDOM_HALF"` or `"ENGLISH_HALF"` or `"CHINESE_HALF"` to determine which half of a new card is shown first.

The code for flash cards has nothing novel in it. We get data from the client browser and use JSON to create an object from the form data, and store the set of flashcards as a JSON string. 

