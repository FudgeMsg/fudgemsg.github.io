---
layout: doc
title: Releases
subtitle: Releases
---

The Fudge Messaging project consists of a number of different sub-projects, which are released independently
(usually just by pushing to the Github repositories). However, we target globally synchronized release points
for ease of development to ensure that functionality is in sync.

#### Release v0.3

Released 2011-03-07.

The v0.3 release was primarily a roll-up of fixes.

* Fudge now uses UTF-8 for Text encoding (previously modified UTF-8)

The binary encoding format is considered stable, although additional types may be added to the standard set.

* Java: [zip](http://dist.fudgemsg.org/java/dist/fudge-java-0.3.zip),
[tar.bz2](http://dist.fudgemsg.org/java/dist/fudge-java-0.3.tar.bz2)
* Proto: [zip](http://dist.fudgemsg.org/java/dist/fudge-proto-0.3.zip),
[tar.bz2](http://dist.fudgemsg.org/java/dist/fudge-proto-0.3.tar.bz2)


#### Release v0.2

Released 2010-02-25.
Release of [Fudge Proto](fudge-proto.html), support for [C applications](c-development.html),
support for streaming, and nearly finalized binary data format.

This release focused on making the project more developer friendly.
There are a number of enhancements that support this effort:

* [Fudge Proto](fudge-proto.html) was written and released for 0.2.
* Java and C# reference implementations have support for streaming APIs for working with fields
(so that you don't have to decode the whole message in order to process data encoded using Fudge).
* Support for registering object encoders and decoders was done (for a JAXB-like use case) for Java and C#.
* The [C Reference Implementation](c-development.html) was written.

In addition, with the exception of moving from Modified UTF-8 to true UTF-8, the binary encoding format
is now *stable* with the 0.2 release. If you're not attempting to encode data (such as unicode {{null}}) that doesn't
encode differently, you won't notice any change in the binary encoding specification after this release.

* Java: [zip](http://dist.fudgemsg.org/java/dist/fudge-java-0.2.zip),
[tar.bz2](http://dist.fudgemsg.org/java/dist/fudge-java-0.2.tar.bz2),
[Fedora Core 11 rpm](http://dist.fudgemsg.org/java/dist/fudge-java-0.2-beta1.fc11.noarch.rpm)
* C#: [zip](http://dist.fudgemsg.org/csharp/dist/fudge-csharp-0.2.zip)
* C: [Windows zip](http://dist.fudgemsg.org/c/dist/fudge-c-msvc-0.2.zip),
[Generic tar.bz2](http://dist.fudgemsg.org/c/dist/fudge-c-0.2.tar.gz),
[Fedora Core 11 rpm](http://dist.fudgemsg.org/c/dist/fudge-c-0.2-beta1.fc11.src.rpm)
* Proto: [zip](http://dist.fudgemsg.org/java/dist/fudge-proto-0.2.zip),
[tar.bz2](http://dist.fudgemsg.org/java/dist/fudge-proto-0.2.tar.bz2),
[Fedora Core 11 rpm](http://dist.fudgemsg.org/java/dist/fudge-proto-0.2-beta1.fc11.noarch.rpm)


#### Release v0.1

Released late 2009.
The first reference implementation in Java and C#.
