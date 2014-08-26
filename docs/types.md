---
layout: doc
title: Types
---

Fudge allows for different types to interpret different values in a data stream.

In particular, the available types in a particular installation of Fudge may be different than any other installation,
and may in fact be runtime-extensible in dynamic systems. While primitive types _should_ have fast-path codings,
all types could be handled by extensible type classes.

### Pre-Defined Fudge Types

These types are part of the built-in Fudge type dictionary, and must be handled by any
[Fudge Encoding Specification](specification.html)-compliant system.

| Type ID (Decimal) | Type ID (Binary) | Fixed Width | Description | Java Type |
| 0 | 0000 0000 | Yes (0) | Unsized indicator value |  [IndicatorType](type-indicator.html) |
| 1 | 0000 0001 | Yes (1) | Boolean | {{boolean}}|
|  2 | 0000 0010 | Yes (1) | signed 8-bit integer | {{byte}}|
| 3 | 0000 0011 | Yes (2) | signed 16-bit integer | {{short}}|
| 4 | 0000 0100 | Yes (4) | signed 32-bit integer | {{int}}|
| 5 | 0000 0101 | Yes (8) | signed 64-bit integer | {{long}}|
| 6 | 0000 0110 | No | byte array | {{byte[]}}|
| 7 | 0000 0111 | No | Array of signed 16-bit integer | {{short[]}}|
| 8 | 0000 1000 | No | Array of signed 32-bit integer | {{int[]}}|
| 9 | 0000 1001 | No | Array of signed 64-bit integer | {{long[]}}|
|10 | 0000 1010 | Yes (4) | 32-bit floating point|{{float}}|
| 11 | 0000 1011 | Yes (8) | 64-bit floating point|{{double}}|
|12 | 0000 1100 | No | Array of 32-bit floating points | {{float[]}}|
|13 | 0000 1101 | No | Array of 64-bit floating points | {{double[]}}|
|14 | 0000 1110 | No | UTF-8 encoded string | {{String}}|
|15 | 0000 1111 | No | Embedded Fudge Message | {{FudgeMsg}}|
|16 | 0001 0000 | N/A | Currently Unallocated |  |
|17 | 0001 0001 | Yes (4) | byte array of length 4 | {{byte\[4\]}}|
|18 | 0001 0010 | Yes (8) | byte array of length 8 | {{byte\[8\]}}|
|19 | 0001 0011 | Yes (16) | byte array of length 16 | {{byte\[16\]}}|
|20 | 0001 0100 | Yes (20) | byte array of length 20 | {{byte\[20\]}}|
|21 | 0001 0101 | Yes (32) | byte array of length 32 | {{byte\[32\]}}|
|22 | 0001 0110 | Yes (64) | byte array of length 64 | {{byte\[64\]}}|
|23 | 0001 0111 | Yes (128) | byte array of length 128 | {{byte\[128\]}}|
|24 | 0001 1000 | Yes (256) | byte array of length 256 | {{byte\[256\]}}|
|25 | 0001 1001 | Yes (512) | byte array of length 512 | {{byte\[512\]}}|
|26 | 0001 1010 | Yes (4) | Reserved for [Date](type-datetime.html) | Currently unspecified|
|27 | 0001 1011 | Unknown | Reserved for [Time](type-datetime.html) | Currently unspecified|
|28 | 0001 1100 | Yes (12) | Reserved for [combined Date and Time](type-datetime.html) | Currently unspecified|

### Type Reductions
To maximise efficiency of a Fudge message, types can be reduced by a system. The following reductions must always take place:

|Type |Reduce to|If |
|16-bit integer|8-bit integer|Value is within smaller range|
|32-bit integer|8- or 16- bit integer|Value is within smaller range|
|64-bit integer|8-, 16- or 32- bit integer|Value is within smaller range|
|byte array|fixed length byte array|Length matches one of the fixed lengths|
|combined date and time|date|If the precision level does not include a time element|

Additional reductions are possible for some types, but their use is optional as there may be a high
overhead of calculating if the conversion is possible or performing the conversion:

|Type |Reduce to|If |
|array of 16-bit integers|array of 8-bit integers|If the largest value in the array is within the 8-bit range|
|array of 32-bit integers|array of 8- or 16- bit integers|If the largest value in the array is within the smaller range|
|array of 64-bit integers|array of 8-, 16- or 32- bit integers|If the largest value in the array is within the smaller range|
|32-bit floating point|8- or 16- bit integer|If the floating point value can be exactly represented by a smaller integer (e.g. 0)|
|64-bit floating point|8-, 16- or 32- bit integer|If the floating point value can be exactly represented by a smaller integer (e.g. 0)|
|64-bit floating point|32-bit floating point|If the floating point value can be exactly represented by the lower precision type|
|array of 32-bit floating point|array of 8- or 16- bit integers|If all elements can be exactly represented by a smaller integer|
|array of 64-bit floating point|array of 8-, 16- or 32- bit integers, or array of 32-bit floating point|If all elements can be exactly represented by a smaller type|
|string|integer or floating point type|If the string contains a base-10 number within the range of an integer type, or a IEE754 ASCII floating point value that can be exactly represented by the 32- or 64- bit floating point type|

Even if these reductions are not implemented, an implementation should provide utility methods or other type
conversion mechanism to expand received data to the type required or expected by an application.

### Runtime-Extended Types
Fudge is designed to have an extensible type system, so applications can add additional types for application
or domain specific entity types. When processing Fudge-encoded data, where a Fudge implementation does not have
the ability to process a particular type, the data will still be maintained so that it can be passed onto
additional systems without loss of data.

To allow scope for the expansion of the standard Fudge types listed above while ensuring application compatibility,
any type extensions must be allocated from 255 (1111 1111) downwards. Any new types added to the standard
will be allocated from 29 upwards.

#### Runtime-Extended Fixed-Width Types
While reference implementations may allow runtime-extended fixed width types, because Fudge doesn't support
specifying the size of a fixed-width type in the field header, the remainder of the message will be unhandled
by any intermediary processing system that doesn't have support for that fixed-width type, and therefore
should be avoided at all costs.

There are two primary reasons for this:

1. The existing set of fixed-width types supports every major binary size up to 512 bytes, meaning that almost all
well-understood fixed width data formats can be handled - GUID, IPv6 address, RSA key, DSA key, SHA-1 digest, MD5 digest
2. As sizes get beyond these known values, the added burden of modelling them as a variable-width field is trumped
by the size of the data payload itself (so adding 2 bytes to encode the length of a 1024 byte field is inconsequential).

In general, well written Fudge-based systems rely on convention rather than specification to determine the mapping.

Therefore, if you are thinking of extending the type system with a new fixed-width type, you should consider the following:

* Could you accomplish the same goals with a convention-based approach on top of one of the existing fixed-width types?
* Would the added burden of a variable-width type be significant enough to cause performance problems?
