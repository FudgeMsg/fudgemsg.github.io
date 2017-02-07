---
layout: default
title: Fudge Messaging - Home page and Documentation
subtitle: The FudgeMsg project
---

This was the home page of the Fudge Messaging project.

**Fudge is no longer actively maintained.**

OpenGamma, the primary sponsors of Fudge, no longer use or maintain Fudge. This page is retained for historical reasons.

-----

[Fudge](docs/acronym.html) is a binary messaging system designed to scale from ad-hoc messages to long-term persistence.

In addition, the major reference implementations allow for the use of Fudge Messages (as code-level objects)
to act as a meta-encoding scheme, allowing translation on the fly to-from XML, JSON, or any other hierarchical format
for which translation utilities have been provided.

### Basics of Fudge Messages

There are a number of well-known messaging systems available. Some focus only small messages, using an external schema.
Others focus on self-description, where the "tag name" is included against each data item.
Fudge is different because it allows both.

A Fudge message is **binary encoded**. There is a detailed [specification](docs/specification.html) outlining the format.
Binary has been chosen over textual formats like XML or JSON to obtain a more compact representation of the data
for the network.

A Fudge message is **hierarchical**. Our view of the world, as adapted to programming languages, is of high-level
structures holding lower level detail. Fudge is structured with messages containing sub-messages,
creating a nested structure of arbitary depth and complexity.

A Fudge message is **field-based**. Each message or sub-message is a partly ordered collection of fields.
The field stores the data item, plus a minimum of two bytes of meta-data.

A Fudge message is **typesafe**. [Types](docs/types.html) add meaning and safety to data.
This is especially useful when storing data for longer periods of time or when receiving data without a schema.
Each field uses a single byte to store the type.
Fudge defines a core set of about 30 types and this can be extended if desired.

A Fudge message is **self-describing** in a flexible way. Each field in the message can be described by a name,
ordinal (number), both or neither. For example, the names might be used in long-term storage where space is less
of an issue, while a compact message would use the ordinal, or perhaps neither (relying on the order of fields).
The ability to process the message without a schema if desired is a key feature.

A Fudge message is **stand-alone**. Fudge has been designed for message-passing, such as in Remote Procedure Calls (RPC)
or Message Oriented Middleware (MOM). It is less suited for streaming data. Because it simply specifies the message
content, it can be used with HTTP, JMS, AMQP and many other underlying transports.

### Project Status
Fudge is in active use. Key components are:

* A binary [Encoding Specification](docs/specification.html) that defines how Fudge data is to be encoded;
* A [Java](docs/java-development.html) reference implementation.
* A [C#/.Net](docs/csharp-development.html) reference implementation.
* A [C](docs/c-development.html) reference implementation.
* A [C++](docs/cpp-development.html) implementation.
* A [Google Protocol Buffers](docs/fudge-proto.html) compatibility library.
* Conversion to [XML](docs/xml.html) and [JSON](docs/json.html) is supported.

You can track the precise status in our [Releases](docs/releases.html) page.


### Why Should I Use Fudge?

Fudge is primarily useful in situations where you have:

* Data exchanging between nodes in a distributed system; where
* You want to be able to do meaningful work with data without knowing at compile time the precise message format; and
* Performance and message size are important; or
* A requirement to encode data and translate between efficient binary encodings, and more accepted text encodings
(XML, JSON) depending on the communications channel.

In a case where you have perfect control over all nodes, and can externally specify (though an API document or
something similar) the schema for each type of message channel, you might be better off using something like
[Google Protocol Buffers](http://code.google.com/apis/protocolbuffers/) or [Thrift](http://incubator.apache.org/thrift/).


### Project Details
* [License](docs/license.html) : Fudge libraries are offered under the [APLv2](http://www.apache.org/licenses/LICENSE-2.0.html).
* [Copyright](docs/copyright.html) : Currently, all copyright to Fudge libraries is held by [OpenGamma](http://www.opengamma.com/) Inc.
* Sponsorship : The Fudge Messaging project was sponsored by [OpenGamma](http://www.opengamma.com/) but is no longer maintained.
* Issue tracking : GitHub issues [Java](https://github.com/FudgeMsg/Fudge-Java/issues),
[Proto](https://github.com/FudgeMsg/Fudge-Proto/issues), [CSharp](https://github.com/FudgeMsg/Fudge-CSharp/issues),
[Historic](/jira/jira.html).
* Website : Edit this website at [GitHub](https://github.com/FudgeMsg/fudgemsg.github.io).

