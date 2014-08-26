---
layout: doc
title: Fudge-Proto syntax
---

`.proto` files are plain text files with tabs, spaces, carriage-return and line-feed characters all considered white-space.
**Comments** can be included anywhere that white-space would be valid using C-style notation. For example:

<pre>
/* This is a
multi-line
comment */

// this is a single-line comment
</pre>

A documentation block comment `/** .. */` is also defined and can be used to prefix an element with documentation
that will be carried through to the target language. This is not currently implemented, and such comments
are ignored at the moment. For example:

<pre>
 /* Copyright statement
    ignored by the code generator */

 /**
  * Information about the message that will be present
  * in any resulting Java or C# code for use by that
  * languages documentation tools.
  */
 message MyMessage {
   ...
 }
</pre>

Any **string literals** are quoted using " characters, with \ as an escape for embedded ", \ and control
characters such as line-feeds.

An **Identifier** may contain any alphanumeric characters or underscores, but most not start with a digit.
The current implementation of the tools reserves all of the type and structure keywords, so these
cannot be used as identifiers. Identifiers should not start with the letters `fudge` as code generated
by the tools will use this prefix for temporary variables. Identifiers are case sensitive.


### Top Level Elements

The `.proto` files are block based using C-style `{` and `}` sequences to contain block contents.
Elements within a block are `;` terminated unless they are a block element.


#### Messages

A message definition can occur at file level or within a message. A message definition consists of:

<pre>
 message &lt;message-identifier> [extends &lt;base-message-identifier>] {
   &lt;message-definitions>
 }
</pre>

A top-level message definition may also include one or more taxonomy references to verify the field names
used are within one or more previously agreed taxonomies, but this functionality is not currently complete
within the tools. The `extends` clause is used to specify [Message Inheritance](fudge-proto-inheritance.html).

Message definitions can consist of fields, nested message definitions, and enumerations.


#### Field definition

A field definition consists of:

<pre>
 [&lt;modifiers>] &lt;field-type> &lt;identifier> [&lt;ordinal>] [&lt;default>];
</pre>

Field modifiers include the following:

| modifier | meaning |
| mutable | Field can be modified after object/record construction (if the target language allows this). This is the default and cannot be used with `readonly`. |
| optional | Field can be omitted from the Fudge message, or from object/record construction. This may be represented as `null` or an uninitialised flag if the target supports it. This is the default and cannot be used with `required`. |
| readonly | Field cannot be modified after object/record construction (if the target language allows this). Cannot be used `mutable`. |
| repeated | Field can appear multiple times in the message. The target language may represent this using a list or array type containing each repeated value. |
| required | Field must be present in the Fudge message or passed to the construction mechanism. Cannot be used with `optional`. When used with `repeated` means the field must be present at least once |

Field types include the [built-in types](fudge-proto-types.html), identifiers for enumerations and other messages.

The field identifier must be unique within a message, and will be used within the Fudge message.
The code generated output will preserve this name in the style of the target language - for example
there may be prefix/suffix or case conventions to follow.

Fields may include an ordinal value, for example:

<pre>
 required string name = 1;
</pre>

Fields may include a default value, for example:

<pre>
 required string name [default = "anonymous"];
</pre>

Fields marked as `required` will result in an error if a message does not contain the field. `Required`
fields also translate to mandatory parameters in a target language. Specifying a default value relaxes this
constraint to a more simple not `null` if that is appropriate to the target language
(where `null` is **not** the same as the empty string).


#### Nested messages

<pre>
 message Outer {
   message Foo { // A
     ...
   }
   required Foo foo1; // refers to Foo-A
   required Middle.Foo foo3; // refers to Foo-B
   required Bar bar1; // not valid - Bar is not in scope
   required Middle.Bar bar2; // valid - explicit reference
   message Middle {
     message Foo { // B
     ...
   }
   message Bar {
     ...
   }
   required Foo foo1; // refers to Foo-B
   required Outer.Foo foo2; // refers to Foo-A
   required Bar bar1; // valid - Bar is in this scope
   required Outer.Middle.Bar bar2;
   // fully qualified name also valid
  }
}
</pre>


#### Language specific data

Extra information can be embedded within a message definition that will be passed to the code generator
for a given language. For example:

<pre>
 message Foo {
   ... message definitions

   binding &lt;language-identifier> {
     &lt;option-identifier> &lt;option-value>;
     ...
   }
 } 
</pre>

Any number of language options can be included within a definition. Any not recognised by the active code
generator will be ignored. Refer to the documentation for a target language code generator for
details of the available options:

* [Java code generator options](fudge-proto-java.html)


### Enumerations

An enumeration is provided in addition to the basic scalar types Fudge supports.
The underlying message encoding will use an integral representation for the value, but this allows symbolic
names to be specified that are then available within a target language to make application code using the
messages easier to read and write. For example:

<pre>
 enum PhoneType {
   HOME;
   MOBILE;
   WORK;
 }
</pre>

The underlying integer values will be allocated from 0 in the order listed in the `.proto` file
Explicit mappings can be given by adding a `= number` clause. Any items which do not explicitly set a value
will take the next available integer after the last explicitly set one. For example:

<pre>
 enum EnumExample {
   FIRST = 10; // explicit value of 10
   SECOND; // implicit value of 11
   THIRD = 9; // explicit value of 9
   FOURTH; // implicit value of 12 (first available integer)
 }
</pre>

An enumeration can be specified at a top level, or if its scope is limited within the message that uses it.
The same rules apply as for nesting messages - refer to the example above for detailsIf a target language has
a type system that supports enumerations, these will be used. Otherwise the field will be exposed as an integral
type with symbolic constants defined that should be used within code.


### Taxonomies

A taxonomy is defined with similar syntax to enumerations, for example:

<pre>
 taxonomy TaxonomyExample {
   name = 1;
   email = 2;
   telephone = 3;
 }
</pre>

As with enumerations, any symbolic names omitting the ordinal will take the value of the next available integer
(the first being 1 as 0 is often used by an application for serialization directives).
Alternatively, a taxonomy can be constructed implicitly by importing field names (and ordinal values if set)
from existing messages. For example:

<pre>
 message Foo {
   string name;
   string email;
 }
 message Bar {
   string name;
   string telephone;
 }
 taxonomy TaxonomyExample {
   import Foo;
   import Bar;
 }
</pre>

Would create give a taxonomy containing three elements - `name = 1`, `email = 2`, and `telephone = 3`.
When taxonomies are built implicitly, the code generator tools should be used to generate a `.proto` file for
the resulting taxonomy to document the assigned ordinals in a system independent manner.


### Namespaces

The top definition level is referred to as a namespace. The default namespace is the empty string.
An application specific namespace can be specified using the `namespace` construct.
A namespace is a dot-seperated list of identifiers. Any of the top level constructs can be embedded within
a namespace - the fully qualified name includes all outer namespaces. For example:

<pre>
 namespace org.fudgemsg.proto {

   message Foo {
     // full name of this message is org.fudgemsg.proto.Foo
   }

   namespace bar {
     message Foo {
     // full name of this message is org.fudgemsg.proto.bar.Foo
     }
   }

 }
</pre>

Target language support for grouping elements into compilation units such as namespaces in .NET languages,
or packages in Java will be used where possible. Where such features are no available, the full identifiers
are converted to a representation that is valid in the global typespace of the target language.

