## Introduction

 Once upon a time there was EBCDIC and ASCII... Actually, it was never that simple and has just become more complex over time. There is light on the horizon, but some estimates are that it may be 50 years before we all live in the daylight on this!

Early computers were developed in the english-speaking countries of the US, the UK and Australia. As a result of this, assumptions were made about the language and character sets in use. Basically, the Latin alphabet was used, plus numerals, punctuation characters and a few others. These were then encoded into bytes using ASCII or EBCDIC.

The character-handling mechanisms were based on this: text files and I/O consisted of a sequence of bytes, with each byte representing a single character. String comparison could be done by matching corresponding bytes; conversions from upper to lower case could be done by mapping individual bytes, and so on.

There are about 6,000 living languages in the world (3,000 of them in Papua New Guinea!). A few languages use the "english" characters but most do not. The Romanic languages such as French have adornments on various characters, so that you can write "j'ai arrêté", with two differently accented vowels. Similarly, the Germanic languages have extra characters such as 'ß'. Even UK English has characters not in the standard ASCII set: the pound symbol '£' and recently the euro '€'

But the world is not restricted to variations on the Latin alphabet. Thailand has its own alphabet, with words looking like this: "ภาษาไทย". There are many other alphabets, and Japan even has two, Hiragana and Katagana.

There are also the hierographic languages such as Chinese where you can write "百度一下，你就知道".

It would be nice from a technical viewpoint if the world just used ASCII. However, the trend is in the opposite direction, with more and more users demanding that software use the language that they are familiar with. If you build an application that can be run in different countries then users will demand that it uses their own language. In a distributed system, different components of the system may be used by users expecting different languages and characters.

*Internationalisation* (i18n) is how you write your applications so that they can handle the variety of languages and cultures. 
*Localisation* (l10n) is the process of customising your internationalised application to a particular cultural group.

i18n and l10n are big topics in themselves. For example, they cover issues such as colours: while white means "purity" in Western cultures, it means "death" to the Chinese and "joy" to Egyptians. In this chapter we just look at issues of character handling. 

