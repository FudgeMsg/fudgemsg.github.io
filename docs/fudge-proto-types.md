---
layout: doc
title: Fudge-Proto types
---

The `.proto` tools support the basic types supported by Fudge.
These are listed below, with the mappings used in the target languages:

|proto keyword |alternative keywords | description | Java type | C# type |
| `bool` | `boolean` | boolean | `boolean` or `Boolean` | ? |
| `byte` | `int8` | 8-bit signed integer | `byte` or `Byte` | ? |
| `double` | | double precision floating point | `double` or `Double` | ? |
| `date` | | date only | `javax.time.calendar.DateProvider` | ? |
| `datetime` | | combined date and time | `javax.time.calendar.DateTimeProvider` | ? |
| `float` | | single precision floating point | `float` or `Float` | ? |
| `indicator` | | indicator flag | `boolean` or `Boolean`, true to indicate presence | ? |
| `int` | `int32` | 32-bit signed integer | `int` or `Integer` | ? |
| `long` | `int64` | 64-bit signed integer | `long` or `Long` | ? |
| `message` | | arbitrary nested message | `org.fudgemsg.FudgeFieldContainer` | ? |
| `short` | `int16` | 16-bit signed integer | `short` or `Short` | ? |
| `string` | | character string | `String` | ? |
| `time` | | time of day | `javax.time.calendar.TimeProvider` | ? |

Arrays of these can be specified using a `[]` suffix which may optionally include the required length,
e.g. `byte[16]`. When lengths are specified, it should not be possible to set the field to an array of
different length, and a Fudge message containing such arrays will be discarded as invalid.
There is no limit on the number of dimensions.

Arrays are encoded using one of the built in array types if possible.
Otherwise they are encoded as a Fudge sub-message.
The sub-message contains repeated fields with no ordinals or names that contain the array elements.
These may in turn be sub-messages, the Fudge specification puts no limit on how deep these may be nested, so
there is no limit on the number of dimensions that can be specified. Dimensions are read from left to right,
so a type of `string[x][][y]` would be encoded as a sub-message containing `x` fields which are sub-messages
containing an arbitrary number of fields, which are sub-messages containing `y` fields of type `string`.

A code generater might expose a repeated field as an array, but arrays are not the same as a repeated field, for example:

<pre>
 message Example1 {
   string[] foo;
 }
 message Example2 {
   repeated string foo;
 }
</pre>

The two messages above are not the same at the protocol level, even if a code generator generates
the same interface to each. The first will be encoded as a message containing a single field, foo, which
is a sub-message containing all elements of foo. The second will be encoded as a message containing a
repeated field for each value of foo.
This is in keeping with the [Serialization Framework](serialization-framework.html) and gives
a more compact encoding.

When the `message` keyword is used to define an arbitrary nested message, it must be prefixed with at
least one modifier (e.g. `optional` or `required`) to distinguish it from a nested message type definition.
This is a limitation of the current parser which will hopefully be removed in a future release.
