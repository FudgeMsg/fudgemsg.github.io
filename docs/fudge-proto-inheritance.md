---
layout: doc
title: Fudge-Proto inheritance
---

A message definition uses the `extends` clause to specify the base message. For example:

<pre>
 message Foo {
   string name;
 }
 message Bar extends Foo {
   string email;
 }
</pre>

The resulting message type `Bar` contains all of the fields of its base type(s) in addition to its own, 
and can be used wherever a message of type `Foo` is expected. A message may not override or redefine a field 
that is already defined in its base class. A field defined with the same name and type will generate a warning 
and be ignored. A field with the same name and a different type will result in an error.

Multiple inheritance is not currently supported and will generate an error, but may be added in a future 
release if there is clear demand for it. For example:

<pre>
 message Foo {
   string name;
   string email;
 }
 message Bar {
   string name;
   string telephone;
 }
 message Combined extends Foo, Bar {
   string id;
 }
</pre>

The message type `Combined` contains four fields `name`, `email`, `telephone`, and `id`. 
The fudge encoding specification allows arbitrary ordering of these within a message. 
The message type `Combined` can be used wherever a `Foo` or `Bar` is expected. Fields with the same name 
(and ordinal if specified) and same type in the base message types are accepted as in the example above. 
Fields with the same name but different ordinals, or different types will result in an error.
