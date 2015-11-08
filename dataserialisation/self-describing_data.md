# Self-describing data

Self-describing data carries type information along with the data. For example, the previous data might get encoded as

    table
        uint8 3
        uint 2
    string
        uint8 4
        []byte fred
    string
        uint8 10 
        []byte programmer
    string
        uint8 6 
        []byte liping
    string
        uint8 7 
        []byte analyst
    string
        uint8 8 
        []byte sureerat
    string
        uint8 7
        []byte manager
    

Of course, a real encoding would not normally be as cumbersome and verbose as in the example: small integers would be used as type markers and the whole data would be packed in as small a byte array as possible. (XML provides a counter-example, though.). 
However, the principle is that the marshaller will generate such type information in the serialised data. The unmarshaller will know the type-generation rules and will be able to use this to reconstruct the correct data structure. 