---
layout: doc
title: Serialization
---

This document is in a draft state and will change shortly. 
A couple of flaws have already been identified in the approach proposed below. 
If you have any further comments, please contact us.

This section describes a standard mechanism for encoding and decoding high-level language objects to 
and from Fudge messages. This is a recommended convention on top of the main Fudge specification. 
The main principles that we're trying to follow:

* Keep the data structure close to the object structure, but if in conflict design for the message.
* Make interoperability between platforms the default - i.e. encourage people to avoid things like platform-specific namespaces on types.
* Support [evolvability of data|#evolvability] natively within the framework, don't make each developer have to bolt this on.
* Keep it easy to use - both for developers hand-coding message/object conversions and for the automated [code generator](fudge-proto.html).
* Don't restrict programming patterns, in particular don't prevent immutable objects.

### Definitions

This part of the specification can be read in the context of any high-level language for which a 
Fudge implementation exists. Terminology can vary between languages so we will use terms as defined below.

| Our term | Definition |
| field | A named attribute, member of property within an object. A field holds a value. Depending on the language it may hold a reference, pointer, or content of another object |
| object | A data construct containing a group of fields. Objects may contain no fields. Objects may contain other objects. |
| reference | Any field is a reference if it is a way of directly or indirectly accessing the containing object, or another object's fields. Depending on the language it may be a pointer, a handle, or the full content of the referenced language. |
| type | A set of fields that objects of that type will hold. A class in C# or Java languages. |

h2. Basic object representation

Every object is represented by a Fudge message. The message field with ordinal 0, or labelled with the 
empty string, is used to indicate the object type. Object fields are represented by further message fields.

| Field | Representation |
| Primitive member | Field with same name as member |
| List or array | Use native Fudge type if available. Otherwise, represent as a sub-message containing a repeated field with no ordinal or name for the list or array data. These fields would in turn be sub-messages in the case of multi-dimensional arrays. |
| Null | If simple field, omit the field or use Indicator (implementations must always deserialize Indicator as null). If it is an element within a list/array, use the Indicator type. |
| Map | Represent as sub-message with two repeated fields. Fields with ordinal 1 contain keys. Fields with ordinal 2 contain values. |
| Reference  | See below |

Note that an alternative list and array representation could have used repeated fields. 
The representation above gives a more efficient packing for any arrays of length 3 or more 
(or of length 2 or more if the field name is more than a couple of characters) due to not repeating the 
field name or ordinal. It also allows arrays of arrays to be encoded in a consistent fashion.

Note that the map representation chosen gives a more efficient packing than representing as a list of key-value pairs. 
The map is defined here as a list of 2-tuples. Although key/value ordering will be preserved within the Fudge 
message, higher level language objects do not have to treat the map as ordered. 
If a language has native object constructs that correspond to higher orders, they could be encoded in a similar fashion. 
This implies that a set could be encoded, and distinguished from a list, by using fields with ordinal 1 instead of no ordinal. 
An implementation that does not support a set may treat such an encoding as a list or array.

#### References

A reference to another object is written as a sub-message.

NOTE: This specification does not address cyclic references or back-references to previous objects.

#### Types and polymorphism

An object's type is a textual string coming from an application defined namespace that is case sensitive. 
The type must contain only alphanumeric, '_' and '.' characters, starting with a letter. 
These characters are either valid within most high level languages or a simple mapping can convert to and 
from valid type names for the language.

The first instance of the type field is the preferred type for the object. Subsequent values for the type field 
within the message are alternative, ancestral, types from the inheritance tree. An implementation must decode 
the message to an object of the first type that it recognises. The object should retain the original type 
details and any additional fields so that data is preserved if it is reserialised.

An optimisation would be to back-reference previous types to avoid repeatedly writing the same type strings. 
This is not addressed by the specification.

To produce a more efficient stream encoding, the type of an object encoded earlier in the message or earlier 
in a parent message can be referenced, instead of repeating the same type strings. 
This can be done by specifying the type as an integer field with a negative value. 
A value of -1 means the type of the sibling or parent object in the field whichever 
sibling or parent object is at index 1 from the head of the buffer. 
In the event of there being multiple objects in the buffer with a matching type, 
the one closest to the head must be used so that the integral encoding is smallest.


### Layout of serialisation stream

A serialisation stream is itself just a Fudge message, with the following fields:
| Name | Type | Purpose |
| version | integer | Version of the Fudge serialisation framework |
| object | message | Repeated field for one or more serialised objects. |

The stream may contain a single object (with the graph of any objects it references), or a sequence of objects.

#### Example

As an example. assuming we have a class Person, which contains a name, a list of siblings, and an optional Address. 
If we serialise a brother-sister combo (starting with "Bob") we get a serialisation stream that looks like:

<pre>
version : byte = 0
object : FudgeMsg =
0 : string = "Person"
name : string = "Bob"
siblings : FudgeMsg = // Begin inline encoding of the "Shirly" object
0 : byte = -1 // Same type as the "Bob" object
name : string = "Shirly"
siblings : FudgeMsg =
0 : byte = 1 // Reference to the "Bob" object
address : FudgeMsg =
0 : string = "Address"
line1 : string = "Our house"
line2 : string = "In the middle of our street"
</pre>


### Evolvability of data

One of the annoying (read costly) things about distributed systems is when you have to upgrade all parts 
simultaneously because a message structure changes.&nbsp; Even worse if you don't actually own all the 
pieces.&nbsp; Evolvability goes some way to alleviate this pain.

We do this:

* Evolvable messages have a version (integer).&nbsp; This is passed in the serialisation stream.
* Messages can be evolved by adding new fields, and increasing the version number.&nbsp; Types of existing fields 
should not be modified unless there is an automatic conversion between the types.
* Objects into which we are deserialising data are coded against a specific version, and know what version this is.
* If an object receives a message with an _older_ version, it is responsible for upgrading the data with sensible 
defaults if possible.&nbsp;&nbsp; This allows an older client to send data to a newer server (or vice-versa), 
decoupling the version dependency.
* If an object receives a message with a _newer_ version, it stores the additional fields and adds them back when 
later reserialising.&nbsp; This allows a service in the middle to pass through future data without needing modification.

In the last scenario, we need to decide which of two strategies to use for the version of the data:

* The data keeps the future version number.&nbsp; This has the down-side that each message will have to carry the 
version number, increasing data bloat.
* The data reverts back to the service's version number even though it carries the future fields.&nbsp; 
This means that the version for all instances of a type will be the same, and also allows the (future) 
recipient to be aware that it has been processed by an older intermediary (so relationships between the 
old and new fields may need to be reenforced).

My vote at the moment is for the latter.
