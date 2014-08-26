---
layout: doc
title: Fudge encoded Taxonomy
---

The rationale behind [taxonomies](taxonomy.html) within the Fudge system is to allow space-efficient
encoding of messages for fast transfer between components.
Any longer term storage should use the full field names to guard against changes to the taxonomy.
In cases where the taxonomy must be made persistent it should be encoded as a Fudge message containing
string fields to hold the expanded names against the unique corresponding ordinal value.
The containing message envelope must not specify a taxonomy.

For example, the taxonomy

| Field name| Ordinal value |
|  id       |  1            |
|  name     |  2            |
|  email    |  3            |

Would be encoded as the message

| Field prefix | Type | Ordinal (2 bytes) | String length (1 byte) | String character data |
|  96          |  14  |  1                |  2                     |  id                   |
|  96          |  14  |  2                |  4                     |  name                 |
|  96          |  14  |  3                |  5                     |  email                |

The bits in the field prefix indicate the ordinal is included, with no name, and String data described
by 1 width byte prefix. Although the fields are given in numerical order in this example,
the encoding specification allows for arbitrary ordering. Taxonomies must only contain field names that are
less than 256 characters in length, so will always use a 1 byte length encoding.
Ordinal values that are not defined by the taxonomy, or that would map to the empty string must not be encoded.

Note that there are no special indicators to distinguish a message in this form from any other message or require
that it be processed any differently to other messages as part of the encoding system. Nor does the presence of
such a message imply any taxonomy details of subsequent or preceding messages in a stream.
It is the responsibility of a decoding application to interpret the message according to any
[serialization convention](serialization-framework.html) it is sharing with the message origin which may support
the dynamic update of a taxonomy library using messages encoded in this manner.

Note also that other encodings are possible but would rely on the list, array, or map serialization conventions
which imply a fixed ordering of data within the message. The specification does not require the ordering of
uniquely named or numbered fields to be preserved within a message allowing applications more flexibility
in that they can select appropriate arbitrary orderings that might allow more efficient algorithms to be used.
