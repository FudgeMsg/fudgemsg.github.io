---
layout: doc
title: Integer Type
---

Fudge attempts to store integral values in the smallest number of bytes feasible. However, unlike systems which 
use a variable integer encoding system (like Varint-128), Fudge does this at the source code library 
perspective by determining the smallest type that can encode the value provided.

Here's how it works:

* Code attempts to add a field of a large integral type (like int64)
* The library determines whether the field provided can actually be stored using a smaller type (like int16)
* If so, the field is stored using the smaller type
* The type identification stored in the field is actually the smaller type. The fact that an int64 was originally provided is lost.
* Message is then encoded and decoded
* Code requests the field value using a large integral type (like msg.getLong("foo") in Java)
* Library implementation sees that there's an int16 stored, and upconverts to an int64 on the fly.

The primary advantage to this scheme is that the data itself is in a format that directly maps to primitive
chip-level types, and doesn't have to be packed or unpacked from a variable width size.
