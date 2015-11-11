## Mutual agreement

The previous section gave an overview of the issue of data serialisation. In practise, the details can be considerably more complex. For example, consider the first possibility, marshalling a table into the stream

    3
    4 fred
    10 programmer
    6 liping
    7 analyst
    8 sureerat
    7 manager
    

Many questions arise. For example, how many rows are possible for the table - that is, how big an integer do we need to describe the row size? If it is 255 or less, then a single byte will do, but if it is more, then a short, integer or long may be needed. A similar problem occurs for the length of each string. With the characters themselves, to which character set do they belong? 7 bit ASCII? 16 bit Unicode? The question of character sets is discussed at length in a later chapter.

The above serialisation is *opaque* or *implicit*. If data is marshalled using the above format, then there is nothing in the serialised data to say how it should be unmarshalled. The unmarshalling side has to know exactly how the data is serialised in order to unmarshal it correctly. For example, if the number of rows is marshalled as an eight-bit integer, but unmarshalled as a sixteen-bit integer, then an incorrect result will occur as the receiver tries to unmarshall 3 and 4 as a sixteen-bit integer, and the receiving program will almost certainly fail later.

An early well-known serialisation method is XDR (external data representation) used by Sun's RPC, later known as ONC (Open Network Computing). XDR is defined by RFC 1832 and it is instructive to see how precise this specification is. Even so, XDR is inherently type-unsafe as serialised data contains no type information. The correctness of its use in ONC is ensured primarily by compilers generating code for both marshalling and unmarshalling.

Go contains no explicit support for marshalling or unmarshalling opaque serialised data. The RPC package in Go does not use XDR, but instead uses "gob" serialisation, described later in this chapter. 