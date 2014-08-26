---
layout: doc
title: XML
---

Fudge messages may be converted to/from XML to support interchange with non-Fudge compliant systems. 
Default element and attribute names are given here for representing Fudge messages in a verbose 
XML format instead of the compact binary format. A specific implementation may allow these to be overridden 
at runtime to better meet a particular application's requirements. 
The conventions expressed here are intended to provide the most flexibility in working with existing XML 
representations of some data. Unless otherwise specified, any string data or attribute values are written 
in the encoding form of the underlying XML transport (e.g. UTF-8). Any integral data or attribute values 
are written as base-10 decimals.

A Fudge message envelope is written as one XML document. The envelope is the single element within the document, 
by default named `fudgeEnvelope`. This may contain the optional attributes:

| Attribute | Definition |
| processingDirectives | The processing directives byte; a numeric value `0..255`. If omitted, or out-of-range, the value `0` may be assumed. |
| schemaVersion | The schema version byte; a numeric value `0..255`. If omitted, or out-of-range, the value `0` may be assumed. |
| taxonomy | The taxonomy identifier; a signed 16-bit taxonomy identifier, or string containing an application specific means to resolve a taxonomy (e.g. a URL). If out-of-range, no taxonomy may be assumed. |

The presence of a `taxonomy` attribute does not affect the XML message - full field details are always present 
if possible. It is available to allow preservation of an original Fudge message envelope.

Inside the envelope element are elements corresponding to each of the Fudge message fields and optional whitespace 
which is ignored. The order of any fields with the same ordinal and/or field names must be preserved when 
converting to/from XML. Each field is written as an element. If the field has a defined name, 
the name should be converted to a valid XML element name by removing any characters that are 
not [NameChar](http://www.w3.org/TR/REC-xml/#NT-NameChar) and any leading characters that are 
not [NameStartChar](http://www.w3.org/TR/REC-xml/#NT-NameStartChar). 
If the field does not have a name, the required conversion yields an empty string, or conversion
is not desired, a generic element must be written, by default named `fudgeField` (or `fudgeFieldN` where 
*N* is the ordinal index if one is available). The element may contain the optional attributes:

| Attribute | Definition |
| name | Full name of the attribute (e.g. if characters were lost in creating the element name, or no element name was generated) |
| ordinal, or index | Ordinal index of the field; a signed 16-bit integer. If out of range, no ordinal may be assumed. |
| key | Alias for `name`, `ordinal`, or `key`. If it is a signed 16-bit integer, is assumed to be the field ordinal index, otherwise it is assumed to be the field name. |
| type | Type of data, if not implicit from the content and/or encoding of the element. |
| encoding | Data encoding, if not implicit from the content and/or type of the element. |

Field types which do not contain data (e.g. the `Indicator` type) may be written as an empty element. 
Otherwise the data is written as the CDATA content. This content includes any whitespace characters. 
An element representing a non-empty sub-message field must contain one or more inner elements describing 
its fields, and whitespace which is ignored.

Field types can be expressed as a string keyword (not case-sensitive) corresponding to the standard 
[Fudge Proto Types](fudge-proto-types.html), or as the relevant integer from a Fudge type dictionary. 
When converting from an XML document to a Fudge message, no type information, an unrecognised type 
string, or an out-of-range type index means to assume an appropriate type based on the element's content. 
For example, if there are nested elements, the sub-message type should be assumed. 
Otherwise the string type should be assumed.

An encoding type should only be specified if the field data cannot be written correctly (or efficiently) 
in the overall encoding of the XML document using the defaults below. 
For example, if the XML document must be written as 7-bit ASCII a Base-64 encoding may be selected 
for binary data or strings that contain Unicode characters. If no encoding is specified, the defaults are:

| Fudge type ID | Type name | Default encoding(s) |
| 0 | `indicator` | no data present |
| 1 | `boolean` | As case insensitive text: `true` or `false`, `T` or `F`, `on` or `off`. Or as integers: `0` or `1` |
| 2 | `byte` | Signed 8-bit decimal |
| 3 | `short` | Signed 16-bit decimal (no commas or other thousand separators) |
| 4 | `int` | Signed 32-bit decimal (no commas or other thousand separators) |
| 5 | `long` | Signed 64-bit decimal (no commas or other thousand separators) |
| 6 | `byte[]` | Comma separated list of signed 8-bit decimals |
| 7 | `short[]` | Comma separated list of signed 16-bit decimals |
| 8 | `int[]` | Comma separated list of signed 32-bit decimals |
| 9 | `long[]` | Comma separated list of signed 64-bit decimals |
| 10 | `float` | IEEE754 character representation |
| 11 | `double` | IEEE754 character representation |
| 12 | `float[]` | Comma separated list of IEEE754 character representations |
| 13 | `double[]` | Comma separated list of IEEE754 character representations |
| 14 | `string` | Characters from the string |
| 15 | `message` | No character data; the fields of the sub-message as nested elements |
| 17 | `byte[]` or `byte[4]` | Comma separated list of signed 8-bit decimals |
| 18 | `byte[]` or `byte[8]` | Comma separated list of signed 8-bit decimals |
| 19 | `byte[]` or `byte[16]` | Comma separated list of signed 8-bit decimals |
| 20 | `byte[]` or `byte[20]` | Comma separated list of signed 8-bit decimals |
| 21 | `byte[]` or `byte[32]` | Comma separated list of signed 8-bit decimals |
| 22 | `byte[]` or `byte[64]` | Comma separated list of signed 8-bit decimals |
| 23 | `byte[]` or `byte[128]` | Comma separated list of signed 8-bit decimals |
| 24 | `byte[]` or `byte[256]` | Comma separated list of signed 8-bit decimals |
| 25 | `byte[]` or `byte[512]` | Comma separated list of signed 8-bit decimals |
| 26 | `date` | RFC3339 or other common convention |
| 27 | `time` | RFC3339 or other common convention |
| 28 | `datetime` | RFC3339 or other common convention |
| unrecognised | | Human readable and machine parseable string representation of the data, the same as for the `string` type |

