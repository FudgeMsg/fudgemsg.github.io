---
layout: doc
title: JSON
---

Fudge messages may be converted to/from [JSON](http://www.json.org/) to support interchange with, for example,
JavaScript based applications as an alternative to writing a Fudge encoder/decoder within JavaScript. 
A Fudge message envelope is written as a JSON object optionally including attributes from the envelope header, 
and fields from the Fudge message as JSON object fields.

| Field | Definition |
| fudgeProcessingDirectives | The processing directives byte; a numeric value `0..255`. If omitted, or out-of-range, the value `0` may be assumed. |
| fudgeSchemaVersion | The schema version byte; a numeric value `0..255`. If omitted, or out-of-range, the value `0` may be assumed. |
| fudgeTaxonomy | The taxonomy identifier; a signed 16-bit taxonomy identifier, or string containing an application specific means to resolve a taxonomy (e.g. a URL). If out-of-range, an absent taxonomy may be assumed. |

The presence of any of these attributes does not affect the JSON message. 
They are available to allow preservation of an original Fudge message envelope.

Fudge fields are added to a JSON object using either the name or ordinal (written as a base-10 signed integer) 
as the JSON field name. An implementation should allow the user to select which of the two to use 
by preference when converting to/from a Fudge message. If the Fudge field has neither name nor ordinal 
it will be named in JSON as the empty string. When converting a JSON object back to a Fudge message, 
the empty string will be treated as an anonymous field.

The JSON specification states that field names should be unique whereas Fudge encoding explicitly allows 
duplicate field names. An application designer should therefore avoid the use of duplicate names in 
Fudge messages that are intended for use with JSON.
In case of fudge message with duplicate field names, the fudge message will be encoded with additional metadata. 
In such case the resulting json will be two element array rather than object. 
The first element of the array will be mentioned metadata and the second one will be the fudge message itself.

All encodings are implicit, using the following approach (for more details please refer 
to [RFC4627](http://www.ietf.org/rfc/rfc4627.txt?number=4627) which describes JSON literal values in more detail):

| Fudge type ID | Type name | Default encoding(s) |
| 0 | `indicator` | `null` |
| 1 | `boolean` | `true` or `false` |
| 2 | `byte` | Signed 8-bit decimal |
| 3 | `short` | Signed 16-bit decimal (no commas or other thousand separators) |
| 4 | `int` | Signed 32-bit decimal (no commas or other thousand separators) |
| 5 | `long` | Signed 64-bit decimal (no commas or other thousand separators) |
| 6 | `byte[]` | Array of signed 8-bit decimals |
| 7 | `short[]` | Array of signed 16-bit decimals |
| 8 | `int[]` | Array of signed 32-bit decimals |
| 9 | `long[]` | Array of signed 64-bit decimals |
| 10 | `float` | IEEE754 character representation |
| 11 | `double` | IEEE754 character representation |
| 12 | `float[]` | Array of IEEE754 character representations |
| 13 | `double[]` | Array of IEEE754 character representations |
| 14 | `string` | Quoted string (see RFC4627) |
| 15 | `message` | JSON object containing fields from the message using the conventions outlined above |
| 17 | `byte[]` or `byte[4]` | Array of signed 8-bit decimals |
| 18 | `byte[]` or `byte[8]` | Array of signed 8-bit decimals |
| 19 | `byte[]` or `byte[16]` | Array of signed 8-bit decimals |
| 20 | `byte[]` or `byte[20]` | Array of signed 8-bit decimals |
| 21 | `byte[]` or `byte[32]` | Array of signed 8-bit decimals |
| 22 | `byte[]` or `byte[64]` | Array of signed 8-bit decimals |
| 23 | `byte[]` or `byte[128]` | Array of signed 8-bit decimals |
| 24 | `byte[]` or `byte[256]` | Array of signed 8-bit decimals |
| 25 | `byte[]` or `byte[512]` | Array of signed 8-bit decimals |
| 26 | `date` | String containing a RFC3339 date or other common convention |
| 27 | `time` | String containing a RFC3339 time or other common convention |
| 28 | `datetime` | String containing a RFC3339 date and time or other common convention |
| unrecognised | | String containing human readable and machine parseable representation of the data |
