---
layout: doc
title: Fudge-Proto
---

While Fudge as a project targets the low-level encoding of hierarchical, self-describing, binary data, we realize
that the most common use case for this type of framework is to pass objects around in a distributed system.
Fudge Proto seeks to solve this problem, by allowing you to:

* Specify the schema for data to be encoded in Fudge messages
* Automatically generate language-specific classes for Java and C# that represents that schema
* Support encoding and decoding to and from language-specific classes in an efficient way.

The easiest way to view the Fudge Proto project is that it's a system akin to Google Protocol Buffers or Thrift,
but the data encoding adheres to the Fudge specification (and is thus self-describing).

### The Basics

The content of a Fudge message is defined within a `.proto` file, very similar to basic
[Google Protocol Buffer](http://code.google.com/apis/protocolbuffers/docs/overview.html) (GPB) files.
Although Fudge encoding and GPB are not compatible at the wire level, it is intended that existing GPB
`.proto` files be usable with the proto-Fudge tools with minimal modification.

A basic `.proto` file may contain a taxonomy, message or enumeration definition.
For example, a basic message containing a person's name, date of birth and contact details might be described as:

<pre>
message Person {
  required string name;
  optional date dateOfBirth;
  optional repeated emailAddress;
  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }
  message PhoneNumber {
    required PhoneType type;
    required string number;
  }
  optional repeated PhoneNumber telephoneNumber;
}
</pre>

The main difference from the GPB format is that the field ordinals can be omitted. Field ordinals can be
specified using a notation such as `required string name = 1;` for a field definition, but the reduction
of message sizes in this manner is best
handled using a taxonomy to reduce commonly used (or long field names) to 2-byte ordinals.

If targeted to an implementation such as Java, C#, or C++ would result in a class with methods to encode/decode
an instance to/from a Fudge encoded message and access the elements by the symbolic names above.
A language such as C that supports structured data would result in a data structure definition and conventional
functions to perform the Fudge encoding/decoding. Additional methods or functions may be generated for equality
comparisons, hash codes, and string representations.


### Getting Started

To get started you will need the pre-built Jar file, available from [releases](releases.html),
in your class path and use the [Command Line Reference](fudge-proto-command-line.html) or
[Ant](fudge-proto-java.html) if you are working with Java.
You will also need the [Antlr 3](http://www.antlr.org/) Java library.

Or, if you want to get involved with the code, check out the repository
from [GitHub](http://github.com/FudgeMsg/Fudge-Proto).

### Further Reference

* [Syntax Summary](fudge-proto-syntax.html)
* [Message Inheritance](fudge-proto-inheritance.html)
* [Types](fudge-proto-types.html)
* [External References](fudge-proto-references.html)
* [Fudge Proto Command Line](fudge-proto-command-line.html)
* [Java Implementation](fudge-proto-java.html)

