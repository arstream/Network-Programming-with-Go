## Unicode gotcha's

This book is not about i18n issues. In particular we don't want to delve into the arcane areas of Unicode. But you should know that Unicode is not a simple encoding and there are many complexities. For example, some earlier character sets used *non-spacing* characters, particularly for accents. This was brought into Unicode, so you can produce accented characters in two ways: as a single Unicode character, or as a pair of non-spacing accent plus non-accented character. 
For example, U+04D6 CYRILLIC CAPITAL LETTER IE WITH BREVE is a single character. It is equivalent to U+0415 CYRILLIC CAPITAL LETTER IE combined with the breve accent U+0306 COMBINING BREVE. This makes string comparison difficult on occassions. The Go specification does not at present address such issues. 

