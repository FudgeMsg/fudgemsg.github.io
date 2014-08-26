---
layout: doc
title: Fudge-Proto references
---

In some cases you may wish to extend a message type from a class (or other element appropriate to the target language)
that has been written manually instead of generated from a `.proto` file, or reference one as a field type
within a message definition. These are supported using the `extern` keyword in a `.proto` file to
register the identifier with the code generator but not produce any code. For example:

<pre>
 extern message MyBaseMessage;
 extern message MySubMessage;
 extern enum MyEnumeration;

 message Example extends MyBaseMessage {
   required repeated MySubMessage subMessageValue;
   required repeated MyEnumeration enumValue;
 }
</pre>

The code generator will assume that definitions for `MyBaseMessage`, `MySubMessage`, and `MyEnumeration`
equivalent to or compatible with the code it would generate for message and enumeration definitions exist and 
will make appropriate references to them in the generated code. The exact details of how they must be declared, 
or requirements for compatibility are specific to the code generator for the target language.

### Code generators

* [External Java classes](fudge-proto-java.html)
