---
layout: doc
title: Specification
---

### Overview

While the term "Fudge" can be used to mean multiple things, fundamentally it is a set of tools and code which
can be used to generate and consume data which adheres to the Fudge Encoding Specification.

This specification is designed with the following key characteristics:

* **Compactness**. Fudge-encoded data should be as small as reasonable.
* **Hardware Efficiency**. Key data structures should be easy to process in hardware or in embedded systems.
* **Software Efficiency**. Key data structures should be simple to encode and decode in software, and should tax the
CPU as little as possible while doing it.
* **Flexibility**. The same encoding specification should be capable of being used in extremely verbose environments
where self description is more important than size, or in extremely performance critical environments where every byte matters.

The classic example of how this works together (and contrasts with a system like Google Protocol Buffers) is
in the case of integral values:

* Fudge stores them in native network byte order integral format, Google Protocol Buffers stores them in zigzag var-128 encoding.
* Fudge shrinks size by storing values smaller than the specified type in a smaller type.
(for example, if a value of 4 is provided for an int32 type, an int8 will actually be encoded).
This has the desired effect of storing fewer bytes for small integral values, without the CPU overhead of var-128 encoding.
* Fudge allows for field names to be included in the data stream, linked in via a taxonomy, or excluded altogether.
Google Protocol Buffers only allows them to be excluded entirely.
* Key message and field boundaries are aligned on 32 and 16 bytes accordingly.


### Total Overhead
For messages containing fields of a fixed width type, Fudge encoding adds the following overhead:

* Per-message overhead - 8 bytes
* Only Sequence-based Field Identification - 2 bytes per field
* Only Ordinal Field Identification - 4 bytes per field
* Ordinal + 10-char Name Identification - 15 bytes per field

So for example a single message with a single field using only ordinal field identification adds 12 bytes total overhead.


### Taxonomy
For situations where messages should be self-descriptive, but also support a rich taxonomy at runtime with full names
for display and search, a message may specify a double byte taxonomy.

At runtime, then, the Fudge system can locate a description of field ordinal (or sequential number) to name mappings,
and provide these automatically to end-user applications without the overhead of encoding the names of all fields in all messages.

Taxonomies are not intended to be internet scale, which is why only two bytes are provided.
Applications will compose the taxonomies they are using from any publicly specified.
Furthermore, a taxonomy doesn't have to be unique to a particular application or message format: as long as
the ordinals are all unique, a number of different message formats can share the same taxonomy definition.

For more information, see the [Taxonomy](taxonomy.html) page.


### Message Envelope

Encoding a Fudge message involves writing two distinct elements:

* A fixed-size Message Header, which contains processing directives for the message
* A series of fields, terminated by a special field type

The processing directives in the Message Header apply to all fields contained in the message,
whether top-level fields or embedded (sub-message) fields. Therefore, sub-messages are simply encoded as a series of fields.

h3. Message Header
Each message envelope begins with the following sequence:

 | Processing Directives | Schema Version | Taxonomy | Message Size |
 |  1 byte               |  1 byte        |  2 bytes |  4 bytes     |

**Processing Directive** - One byte containing a bit field specifying options for the processing of the message.
This byte is currently entirely for future expansion and to improve the alignment of the fields in the message header,
and is not currently used.

**Schema Version** - One byte indicating the version of the logical schema that was used to generate the message.
This is a field specifically for the use of the encoding application, and is used to signal to receiving applications
which application-specific decoding logic should be used.

**Taxonomy** - Two bytes acting as an identifier for the taxonomy to be looked up in the taxonomy reference table
specific to the message sender.
This is *not* a globally unique reference, and is specific to the taxonomy of the message encoder.

**Message Size** - Four bytes, defining the size, in bytes, of the message.
The message size is the total size of all data in the message, and includes the message envelope itself.


### Field Encoding
A particular field is encoded in the following manner:

 | Field Prefix | Type    | Ordinal  | Name                       | Data      |
 |  1 byte      |  1 byte |  2 bytes |  variable, up to 256 bytes |  variable |

**Field Prefix** - See below.

**Type** - 1 byte indicating the type of data included

**Ordinal** - If provided, a 2-byte *signed* integral ordinal for the specification of the field

**Name** - If provided, a variable encoded (exactly 1 width byte) textual description for the field.

**Data** - Variable or fixed encoded data (determined by the type). Number of bytes of size prefix is given in the header.

#### Field Prefix
The field prefix is a 1-byte header which contains the following bit fields:

 | Bit 7            | Bits 6-5                      | Bit 4        | Bit 3     | Bits 2-0         |
 | 1 if fixed width | Variable Width Size Indicator | 1 if ordinal | 1 if name | Future expansion |
 | 0 if variable    | Variable Width Size Indicator | 0 if not     | 0 if not  |                  |

**Variable Width Size Indicator** - These two bytes will have one of the following values:

 | Value | Meaning                                                          |
 | 00    | Either a fixed width type, or an empty variable width field      |
 | 01    | 1 byte is used to encode the size of the variable width payload  |
 | 10    | 2 bytes is used to encode the size of the variable width payload |
 | 11    | 4 bytes is used to encode the size of the variable width payload |

#### Types
The Fudge specification includes a set of Standard [Types](types.html) that any compliant system must support,
with defined reduction rules so that the smallest possible encoding for the data is used.

#### Sub-Message Encoding

Sub-messages are encoded using the following system:

* In the containing message, a field entry is provided with the `StartFudgeMsg` type.
A name, ordinal, both or neither may be provided as with any other field.
* The `StartFudgeMsg` field is variable-width. The 2-bit variable-width field size indicator will be provided as usual.
* The size of the sub-message is specified as defined by the 2-bit variable-width field size indicator.
* The end of the sub-message is implied by the size sent with the `StartFudgeMsg` field.
* The fields of the sub-message will immediately follow in the data stream

Note that aside from the `StartFudgeMsg` field type header, this is the exact same processing as
for the fields in a Fudge Message Envelope.

#### Variable-width fields

All variable-width data is composed of an N-byte prefix (the number of bytes is either fixed or specified
in the field encoding prefix) coupled with the actual variable data.

#### Text

All text MUST be encoded in UTF-8.

#### Numbers (Endianness)

All data MUST be encoded in Network Byte Order, notably integers and IEEE-751 floating point numbers.


### Field Ordering

The ordering of repeated fields within a message, or sub-message, MUST be preserved by any compliant system
as the ordering can be used to represent sequence within a list or other higher level programming construct.
Fields are considered to be repeated if:

* both do not specify a Name or Ordinal (anonymous fields); or
* both specify a Name and the Names match; or
* both specify an Ordinal and the Ordinals match.

For any other fields, the ordering is not guaranteed to be preserved.
