## Definitions

It is important to be careful about exactly what part of a text handling system you are talking about. Here is a set of definitions that have proven useful.

### Character

A character is a "unit of information that roughly corresponds to a grapheme (written symbol) of a natural language, such as a letter, numeral, or punctuation mark" (Wikipedia). A character is "the smallest component of written language that has a semantic value" (Unicode). This includes letters such as 'a' and 'À' (or letters in any other language), digits such as '2', punctuation characters such as ',' and various symbols such as the English pound currency symbol '£'.

A character is some sort of abstraction of any actual symbol: the character 'a' is to any written 'a' as a Platonic circle is to any actual circle. The concept of character also includes control characters, which do not correspond to natural language symbols but to other bits of information used to process texts of the language.

A character does not have any particular appearance, although we use the appearance to help recognise the character. However, even the appearance may have to be understood in a context: in mathematics, if you see the symbol π (pi) it is the character for the ratio of circumference to radius of a circle, while if you are reading Greek text, it is the sixteenth letter of the alphabet: "προσ" is the greek word for "with" and has nothing to do with 3.14159...

### Character repertoire/character set

A character repertoire is a set of distinct characters, such as the Latin alphabet. No particular ordering is assumed. In English, although we say that 'a' is earlier in the alphabet than 'z', we wouldn't say that 'a' is less than 'z'. The "phone book" ordering which puts "McPhee" before "MacRea" shows that "alphabetic ordering" isn't critical to the characters.

A repertoire specifies the names of the characters and often a sample of how the characters might look. e.g the letter 'a' might look like 'a', 'a' or 'a'. But it doesn't force them to look like that - they are just samples. The repertoire may make distinctions such as upper and lower case, so that 'a' and 'A' are different. But it may regard them as the same, just with different sample appearances. (Just like some programming languages treat upper and lower as different - e.g. Go - but some don't e.g. Basic.). On the other hand, a repertoire might contain different characters with the same sample appearance: the repertoire for a Greek mathematician would have two different characters with appearance π. This is also called a noncoded character set.

### Character code

A character code is a mapping from characters to integers. The mapping for a character set is also called a coded character set or code set. The value of each character in this mapping is often called a code point. ASCII is a code set. The codepoint for 'a' is 97 and for 'A' is 65 (decimal).

The character code is still an abstraction. It isn't yet what we will see in text files, or in TCP packets. However, it is getting close. as it supplies the mapping from human oriented concepts into numerical ones.

### Character encoding

To communicate or store a character you need to encode it in some way. To transmit a string, you need to encode all characters in the string. There are many possible encodings for any code set.

For example, 7-bit ASCII code points can be encoded as themselves into 8-bit bytes (an octet). So ASCII 'A' (with codepoint 65) is encoded as the 8-bit octet 01000001. However, a different encoding would be to use the top bit for parity checking e.g. with odd parity ASCII 'A" would be the octet 11000001. Some protocols such as Sun's XDR use 32-bit word-length encoding. ASCII 'A' would be encoded as 00000000 00000000 0000000 01000001.

The character encoding is where we function at the programming level. Our programs deal with encoded characters. It obviously makes a difference whether we are dealing with 8-bit characters with or without parity checking, or with 32-bit characters.

The encoding extends to strings of characters. A word-length even parity encoding of "ABC" might be 10000000 (parity bit in high byte) 0100000011 (C) 01000010 (B) 01000001 (A in low byte). The comments about the importance of an encoding apply equally strongly to strings, where the rules may be different.

### Transport encoding

A character encoding will suffice for handling characters within a single application. However, once you start sending text *between* applications, then there is the further issue of how the bytes, shorts or words are put on the wire. An encoding can be based on space- and hence bandwidth-saving techniques such as `zip`'ping the text. Or it could be reduced to a 7-bit format to allow a parity checking bit, such as `base64`.

If we do know the character and transport encoding, then it is a matter of programming to manage characters and strings. If we don't know the character or transport encoding then it is a matter of guesswork as to what to do with any particular string. There is no convention for files to signal the character encoding.

There is however a convention for signalling encoding in text transmitted across the internet. It is simple: the header of a text message contains information about the encoding. For example, an HTTP header can contain lines such as

```
Content-Type: text/html; charset=ISO-8859-4
Content-Encoding: gzip
```  

which says that the character set is ISO 8859-4 (corresponding to certain countries in Europe) with the default encoding, but then gziped. The second part - content encoding - is what we are referring to as "transfer encoding" (IETF RFC 2130).

But how do you read this information? Isn't it encoded? Don't we have a chicken and egg situation? Well, no. The convention is that such information is given in ASCII (to be precise, US ASCII) so that a program can read the headers and then adjust its encoding for the rest of the document. 

