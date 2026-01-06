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

| Human readable    | AscII characters are used for the symbols that mark parts of the specification, these are easy to pick out from the text and are used extensively in programming.  The field names and labels are human readable strings. |
| Machine readable  | Computers should be able to parse a BPDS definition string to be able to act on a byte stream encoded with in that format.  This is useful for things like generic highlighters or error checkers. |
| Intuitive         | Someone should be able to look at a BPDS definition string and more or less understand how to interpret the meaning without needing to read a document explaining what BPDS is or how it works. |
| Single line       | A single line should be able to describe a packet.  This keeps the description compact and allows it be added to larger documents. |
| Text based        | Sticking with a text based description means it can be easily copied and embedded in other documents like in source code. |
| Byte based        | Most protocols are byte based and trying to support arbitrary or variable bit protocols would make the standard to complex, so to keep things simple it is restricted to bytes. |

# Terminology
| Field             | A grouping of bytes that form an element of the protocol.  This maybe a single byte or multiple bytes together.  This identifies a part of the message.  For example the length would be considered a field that indicates the length in the message. |
| Literal           | A constant value that must match this value in the byte stream. |
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
| <>                | Field             | Marks the start and end of a field.  A field group multiple bytes together and marks an element in the protocol. | 0x3C 0x3E |
| Literal           | Literal Value     | This byte does not have a name and is just the literal value.  This is a number can uses C number prefixes (0x for hex, 0 for octal, etc) or a string surrounded by quotes ("). |
| =                 | Assigned value    | The field will be have this value (or set of values).  This is the same as a literal but comes after the name of the field. | 0x3D |
| \|                | OR                | Can only be used with values.  When you want to use a set of values instead of just one you place an | between the values and it counts as this value or this other value. | 0x7C |
| :                 | Size              | The number of bytes this field is (if not provided defaults to 1, so “cmd” and “cmd:1” are the same). | 0x3A |
| ...               | Match Any         | This is only used as a ‘Size’ with the size symbol (:).  It matches any number of bytes until it finds a match for the next field.  This size can match 0 bytes. | 0x2E 0x2E 0x2E |
| ()                | Data Type         | What type of data is this field. If you use this, you must also specify the size (:). | 0x28 0x29 |
| "                 | Quote             | A quote that marks the start and end of a string value.  Used with string literals. | 0x22 |


