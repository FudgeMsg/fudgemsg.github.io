---
layout: doc
title: Releases
---

The Fudge Messaging project consists of a number of different sub-projects, which are released independently
(usually just by pushing to the Github repositories). However, we target globally synchronized release points
for ease of development to ensure that functionality is in sync.

#### Release v0.3

Released 2011-03-07.

The v0.3 release was primarily a roll-up of fixes.

* Fudge now uses UTF-8 for Text encoding (previously modified UTF-8)

The binary encoding format is considered stable, although additional types may be added to the standard set.

* [Java downloads](https://github.com/FudgeMsg/Fudge-Java/releases/tag/v0.3)
* [Proto downloads](https://github.com/FudgeMsg/Fudge-Proto/releases/tag/v0.3)


#### Release v0.2

Released 2010-02-25.

This release focused on making the project more developer friendly.
There are a number of enhancements that support this effort:

* [Fudge Proto](fudge-proto.html) was written and released for v0.2.
* Java and C# reference implementations have support for streaming APIs for working with fields
(so that you don't have to decode the whole message in order to process data encoded using Fudge).
* Support for registering object encoders and decoders was done (for a JAXB-like use case) for Java and C#.
* The [C Reference Implementation](c-development.html) was written.

In addition, with the exception of moving from Modified UTF-8 to true UTF-8, the binary encoding format
is now *stable* with the 0.2 release. If you're not attempting to encode data (such as unicode {{null}}) that doesn't
encode differently, you won't notice any change in the binary encoding specification after this release.

* [Java downloads](https://github.com/FudgeMsg/Fudge-Java/releases/tag/v0.2)
* [CSharp downloads](https://github.com/FudgeMsg/Fudge-CSharp/releases/tag/v0.2)
* [C downloads](https://github.com/FudgeMsg/Fudge-CSharp/releases/tag/v0.2)
* [Proto downloads](https://github.com/FudgeMsg/Fudge-Proto/releases/tag/v0.2)


#### Release v0.1

Released late 2009.
The first reference implementation in Java and C#.

* [Java downloads](https://github.com/FudgeMsg/Fudge-Java/releases/tag/v0.1)

