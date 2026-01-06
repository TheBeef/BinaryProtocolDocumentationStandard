“
”

<p align="center">Binary Protocol Documentation Standard</p>
<p align="center">1.0</p>

# Rational
When working with binary protocols the format and details of the documentation have been inconstant and often hard to follow.  The binary protocol documentation standard (BPDS) tries to fix this, using a standard format that is human readable, intuitive, and text based.  The hope is that will improve the readability of the documentation for byte based protocols.

The format is well defined and designed so that a machine can parse a BPDS definition and be able to decode what bytes in a protocol are for what.  It can not interpret the meaning of the byte, but would be able to extract the structure of the packet, knowing how to recognize the start and end of the packet and label the parts.

The BPDS does not try to represent possible states, message responses, the flow of a protocol, handshaking, or how the protocol should be used.  It just tries to explain the structure of a packet of data, not how to work with it.

# Example
We start with an example to give an idea of how this works.

```
<Header=0xFF><Version><Cmd><Len:2><Data:Len><Footer=0x77>
```

This is a generic description of a made up protocol.  You can see that it starts with a header set to the value 0xFF followed by the version and the command, because there is no size provided then these are 1 byte each.  The length is next and the :2 tells us that it's 2 bytes.  The length is followed by the data and we can see the size is a field 'Len' so there will be 'Len' bytes of data (that we can only determine when we read a packet).  The data will be followed by a footer byte that is always set to 0x77. 

So you would expect to see a data stream something like:

```
0xFF 0x01 0x01 0x00 0x08 0x64 0x64 0x10 0x10 0x00 0xFF 0x00 0x00 0x77
```
Documented in Autodoc format (as a C comment):

```
/*******************************************************************************
 * NAME:
 *    Command Protocol
 *
 * SYNOPSIS:
 *    <Header=0xFF><Version><Prop><Cmd><Len:2><Data:Len><Footer=0x77>
 *
 * PARAMETERS:
 *    Head -- Marker used for resyncing.  Always 0xFF.
 *    Version -- What version of the protocol we are using.  Currently only
 *               version 1 is defined.
 *    Prop -- What optional properties are included in this packet.  This is a bit
 *            field where:
 *               0x01 -- Transparency
 *               0x02 -- Fill style
 *               0x04 -- Border
 *    Cmd -- What command we are going to be executing.  See details below
 *           for a description of available commands.
 *    Len -- The length of the 'Data'.  This does not include the 'Footer',
 *           only the data it's self.  This is in big endian.
 *    Data -- The data for this command.  The size will depend on the command.
 *    Footer -- Marker used for resyncing.  Always 0x77.
 *
 * FUNCTION:
 *    This is general packet format for sending commands to the graphic
 *    processor system.
 *
 * REPLY:
 *    Replies will be an acknowledge packet.
 *
 * SEE ALSO:
 *    
 ******************************************************************************/
```

As you can see when used with an Autodoc style comment it is very effective at explaining the protocol and can be included directly with the handling code.

# Goals
The binary protocol documentation standard has a number of goals:
| Goal              | Description |
| ---               | ---         |
| Human readable    | AscII characters are used for the symbols that mark parts of the specification, these are easy to pick out from the text and are used extensively in programming.  The field names and labels are human readable strings. |
| Machine readable  | Computers should be able to parse a BPDS definition string to be able to act on a byte stream encoded with in that format.  This is useful for things like generic highlighters or error checkers. |
| Intuitive         | Someone should be able to look at a BPDS definition string and more or less understand how to interpret the meaning without needing to read a document explaining what BPDS is or how it works. |
| Single line       | A single line should be able to describe a packet.  This keeps the description compact and allows it be added to larger documents. |
| Text based        | Sticking with a text based description means it can be easily copied and embedded in other documents like in source code. |
| Byte based        | Most protocols are byte based and trying to support arbitrary or variable bit protocols would make the standard to complex, so to keep things simple it is restricted to bytes. |

# Terminology
| Term              | Description |
| ---               | ---         |
| Field             | A grouping of bytes that form an element of the protocol.  This maybe a single byte or multiple bytes together.  This identifies a part of the message.  For example the length would be considered a field that indicates the length in the message. || Literal           | A constant value that must match this value in the byte stream. |
| Symbol            | A byte that is recognized as having meaning in the BPDS.  For example < and > mark the start and end of a field. |
| Field name        | The name of a field.  This is alpha numeric, starting with a letter. |
| Label             | When a size refers to a previous field name it is called a label.
| Value             | The value of a field when a literal is used.  This maybe a number or a string. |
| Attribute         | An attribute is a symbol that modifies a field.  An example is the Size (:) attribute. |
| Definition        | The whole BPDS string that defines a matching set of fields. |

# Format
## Symbols
| Symbol            | Name              | Description | Value |
| ---               | ---               | ---         | ---   |
| <>                | Field             | Marks the start and end of a field.  A field group multiple bytes together and marks an element in the protocol. | 0x3C&nbsp;0x3E |
| Literal           | Literal Value     | This byte does not have a name and is just the literal value.  This is a number can uses C number prefixes (0x for hex, 0 for octal, etc) or a string surrounded by quotes ("). | |
| =                 | Assigned value    | The field will be have this value (or set of values).  This is the same as a literal but comes after the name of the field. | 0x3D |
| \|                | OR                | Can only be used with values.  When you want to use a set of values instead of just one you place an \| between the values and it counts as this value or this other value. | 0x7C |
| :                 | Size              | The number of bytes this field is (if not provided defaults to 1, so “cmd” and “cmd:1” are the same). | 0x3A |
| ...               | Match Any         | This is only used as a ‘Size’ with the size symbol (:).  It matches any number of bytes until it finds a match for the next field.  This size can match 0 bytes. | 0x2E&nbsp;0x2E&nbsp;0x2E |
| ()                | Data Type         | What type of data is this field. If you use this, you must also specify the size (:). | 0x28&nbsp;0x29 |
| "                 | Quote             | A quote that marks the start and end of a string value.  Used with string literals. | 0x22 |

### Fields
A field start with a < symbol and ends with a > symbol.  A field is a group of bytes together and makes up a base element in the protocol.  A field includes information about the group such as size, literal values, and type information.

If the field is a literal then it is just the literal and does not use any other attributes expect the OR (|) attribute.  The literal can be a number or a string.  If it’s a number then it uses C number prefixes (0x for hex, 0 for octal, etc) and can be any number of bytes (although going over 8 bytes (64 bit) might make parsers fail).

If it’s a string then it uses quotes around it and will be use a size the same as the string length.  For example <"Dog"|"Fish"> will match a 3 byte string or a 4 byte string.

If the field is not a literal then it starts with the name of the field followed by any attributes.  For example <Start>, <Start:2>, <Start=0xFF>.

#### Examples
| BPDS              | Description |
| ---               | ---         |
| <0x55>            | A literal that must be 55 hex. |
| <0x55\|0xAA>      | A literal that must be 55 hex OR AA hex. |
| <32>              | A literal that must the 32 decimal. |
| <"Cat">           | A literal that must match 0x43, 0x61, and 0x74 |
| <"Cat"\|"Dog">    | A literal that must match 0x43, 0x61, and 0x74 OR 0x44 0x6F and 0x67 |
| \<Start\>         | A field with the name of "Start".  It can be any 1 byte value (the value doesn’t mater only that it is 1 byte long). |
| \<Start:2\>       | A field with the name "Start" that is 2 bytes in length.  The value doesn't mater, just that it is 2 bytes in length. |
| \<Start=0x55\>    | A field with the name "Start" that is 1 byte long and must be the value 55 Hex. |

### Literal Value
A literal value is a constant value that must match.  These are numbers or strings.  Strings are wrapped in quotes.  Numbers use C number prefixes (0x for hex, 0 for octal, etc).

#### Examples
| BPDS              | Description |
| ---               | ---         |
| <0xFF>            | Must match the value 255 |
| <0xFF\|0xEE>      | Can match 0xFF OR 0xEE |
| <"Hello">         | Must match 0x48 0x65 0x6c 0x6c 0x6f |
| <"Hello"\|"Bye">  | Can match 0x48 0x65 0x6c 0x6c 0x6f, OR 0x42 0x79 0x65 |

### Assigned value
The assigned value is the same as a Literal value but with a field name.  The literal value is after a equal sign (=) and follows the same rules as a literal.

#### Examples
| BPDS                      | Description |
| ---                       | ---         |
| <Name=0xFF>               | Field has the name “Name” as must match the value 0xFF |
| \<Start=0xFF\|0xEE\>      | Field has the name “Start” and can match 0xFF or 0xEE |
| <Command="Hello"\|"Bye">  | Field has the name “Command” and can match 0x48 0x65 0x6c 0x6c 0x6f, OR 0x42 0x79 0x65 |

### OR
The or symbol (|) is used to say any literal from a set of literal can be a match.  These can be numbers or strings (but they can not be mixed).  You list all the values you which to accept with a pipe bar between them.  This is valid in assigned values and literal values.

#### Examples
| BPDS                      | Description |
| ---                       | ---         |
| <0x55\|0xAA\|0x00>        | A literal that must be 55 hex OR AA hex OR 00 hex. |
| <Start=0xFF\|0xEE>        | Field has the name “Start” and can match 0xFF OR 0xEE |
| <Command="Hello"\|"Bye">  | Field has the name “Command” and can match 0x48 0x65 0x6c 0x6c 0x6f, OR 0x42 0x79 0x65 |

### Size
The size symbol tells you how many bytes this field uses.  The size symbol uses the colon (:) and must follow a field name.  If the size symbol is not provided then the size of the field will be 1 byte.

This can also be the field name of a previous field. In this case what is being stated is that this field is variable length and the number of bytes to expect comes from this previous field (label).

This can also be set to match any (...) in which case it means that the size of this field is variable and may between 0 and unlimited.  The field is terminated by the next field.  So for example if a size is match any and the next field is a literal 0x0A then all the bytes between this point and the 0x0A fit into this field.  See Match Any below for more info.

#### Examples
| BPDS                      | Description |
| ---                       | ---         |
| \<Len:2\>                 | The length is 2 bytes |
| \<Data:32\>               | This field is 32 bytes long |
| \<Start:2=0xDEAD\>        | The field “Start” is 2 bytes in size and must match the value 0xDEAD |
| \<Other:3="Cat"\>         | The field “Other” is 3 bytes and must match 0x43 0x61 0x74 |
| \<More:Prev\>             | The field “More” uses the value from the previous “Prev” field. |

### Match Any
The match any symbol (...) means match all bytes until the next field is satisfied.

For example if you have <Data:...><0x0A> this matches all chars until a 0x0A (new line) char is found (the new line will not be part of ‘Data’).  So this will match (\n = 0x0A):
| Stream            | Data Field    | 0x0A Field | Description |
| ---               | ---           | ---        | ---         |
| Test\n            | Test          | 0x0A       | The string “Test” will be in data |
| A long string\n   | A long string | 0x0A       | The string “A long string” will be in data |
| \n                |               | 0x0A       | A blank string will be in data |

If the next field is more than 1 byte then all the bytes have to match and will not be part of the field using the match any symbol.

#### Examples
| BPDS                                                  | Description |
| ---                                                   | ---         |
| \<Data:...\>\<0x00\>                                  | A zero terminated string |
| \<CmdNum:...\>\<EndOfCmd="END"\>                      | A string that must end is the string “END”.  So 0x31 0x32 0x45 0x4E 0x44 would end up with a CmdNum field = to “12”. |
| \<Comment:...\>\<EndOfComment="."\>                   | A comment that ends with a period. |
| \<0xFF\>\<Cmd\>\<Data:2\>\<Note:...\>\<0x00\>\<0x77\> | A longer definition with a ‘Note’ field that is variable size terminated by a NULL char. |

