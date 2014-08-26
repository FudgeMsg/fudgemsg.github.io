---
layout: doc
title: Taxonomy
---

The rationale for the inclusion of taxonomies into Fudge is that:

* Working with string-based field names is a desirable feature, because it makes coding and introspecting
far easier than just working with index- or ordinal-based message encoding.
* Including string-based field names in a message is an unnecessary performance hit in a high-performance system.

Thus, taxonomies allow you to do all your coding using field names, but not transmit those when the message
is encoded to a series of bytes. Rather, just a taxonomy ID is transmitted, along with the ordinals matching
those field names. At decoding time, the taxonomy is looked up, and field names are restored.

Taxonomies are not strictly appropriate for long-term persistence of Fudge messages, because the taxonomy
might change over time, or be lost. Therefore, when writing to disk or another persistent message format,
field names should be restored. The taxonomy could be stored using an approach such as the
[Fudge Encoded Taxonomy](taxonomy-encoded.html), however conventional compression techniques
on a storage medium may give a similar space saving.
