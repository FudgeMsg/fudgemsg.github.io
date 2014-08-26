---
layout: doc
title: Fudge-Proto Java
---

The Java code generator creates a `.java` file for each message definition, enumeration and taxonomy it 
finds in the `.proto` files - each map to a class. The namespaces map directly to Java packages, and the 
output files are created in an appropriate directory structure. Anything declared outside of a namespace 
corresponds to the Java default package. Messages and enumerations defined within an outer message are written 
as static inner classes.


### Java command line

Please refer to the [Fudge Proto Command Line](fudge-proto-command-line.html) for full details on
launching the command line `.proto` compiler from the command line. Java specific options are:

| Option | Description |
| -lJava | Selects the Java code generator |
| -Xequals | Generate `equals` methods for value equality of message fields (default is not to) |
| -XfudgeContext=_expression_ | Generates serialization methods that do not require context as a parameter and will use a given global expression specific to the application |
| -XgitIgnore | Write a `.gitignore` in the target folder(s) listing the generated `.java` files |
| -XhashCode | Generate `hashCode` methods hashing the message field values (default is not to) |
| -XtoString | Generate `toString` methods using the commons-lang `ToStringBuilder` (default is not to) |

The method options apply to all output and can be overridden on a per-message basis using the `methods` directive.


### Ant task

The Java code generator supports Ant based build systems. The Ant task is defined in `org.fudgemsg.proto.AntTask`, 
will scan a source folder for `.proto` files and generate `.java` files for any that have changed or that do not 
have a corresponding `.java` file.

<pre>
 &lt;taskdef name="fudgeproto" classname="org.fudgemsg.proto.AntTask" classpathref="lib.path.id" />

 &lt;target name="fudge-proto">
  &lt;fudgeproto />
 &lt;/target>
</pre>

Attributes available for the task are:

| Attribute | Description | Default value |
| destdir | Output folder for `.java` files | src |
| equals | Write `equals` methods in messages | true |
| fieldsMutable | Make fields mutable if neither the `mutable` or `readonly` modifier is given | true |
| fieldsRequired | Make fields required if neither the `required` or `optional` modifier is given | false |
| fudgeContext | Create serialization methods that do not take a context parameter using this default instead | |
| gitIgnore | Write a `.gitignore` file with the generated code | false |
| hashCode | Write `hashCode` methods in messages | true |
| rebuildAll | Ignore timestamps and always process `.proto` files | false |
| searchdir | Additional paths for `.proto` files to be read but not generate `.java` for | |
| srcdir | Source folder for `.proto` files | src |
| toString | Write `toString` methods in messages | false |
| verbose | Write debugging output to stdout | false |

Typically the `.proto` files will be in your source tree with other `.java` files, so the generated ones can be 
written to the same location. The main `javac` task in the Ant script can then declare the Fudge proto 
task a dependency.


### Java specific message definitions

Java specific definitions allow custom code to be written to the generated `.java` files or override global 
behaviours from the command line or Ant settings. Supported options are:

* `body`
* `delegate`
* `fudgeContext`
* `implements`
* `imports`
* `methods`


#### Body

Embeds a source code fragment in the body of a `.java` file. This can be used to include custom attributes 
or methods, for example:

<pre>
 message Foo {
   required string name;
   binding Java {
     body &lt;&lt;&lt;CODE
       public int getNameLength () {
         return getName ().length ();
       }
 CODE;
   }
 }
</pre>

Note that the code will be written within the body of the class definition, so any class names must be fully 
qualified, or imported using the `imports` option.


#### Delegate

Allows the builder design pattern to return a sub-class of the message definition class. 
The nominated delegate class must be a sub-class of the message definition class that has a visible 
constructor (from the perspective of the corresponding `Builder` class) that takes a single parameter of 
the `Builder`. For example, this might be to implement more rigorous business logic constraints:

<pre>
 /* Foo.proto */
 message Foo {
   optional string name;
   binding Java {
     delegate "FooImpl";
   }
 }
</pre>

<pre>
 /* FooImpl.java */
 class FooImpl extends Foo {
   /* package */ FooImpl (Foo.Builder builder) {
     super (builder);
     ... attribute checking statements to validate the construction
   }
 }
</pre>


#### FudgeContext

Normally the `toFudgeMsg` method will require a `FudgeMessageFactory` (or `FudgeSerializationContext` 
if it has external references) parameter. A `fromFudgeMsg` method may also require a `FudgeDeserializationContext` 
if the message has external references. Specifying a default context globally (using the command line) or on
a per-message basis with this option will produce additional methods that do not take this parameter
and use this default instead.

*Note*: this should be used with caution. Relying on a global context may work well for smaller projects but 
may cause issues when scaling to larger environments where multiple contexts might be required 
(e.g. where a node has to communicate with two others, each with different type or taxonomy dictionaries). 
Care must also be taken to avoid the risk of infinite recursion as the tools for detecting and handling cyclic 
message or object graphs is contained within the `FudgeSerializationContext`.


#### Implements

Adds additional `implements` interfaces to the generated class declaration. 
All messages by default will implement `java.io.Serializable`. A generated message definition class cannot 
be abstract, so any methods from additional interfaces will need to be defined in a `body`
option if they will not result from the field definitions. It takes a comma separated list. For example:

<pre>
 message Foo {
   required string name;
   binding Java {
     implements "Comparable&lt;Foo>";
     body &lt;&lt;&lt;CODE
       public int compareTo (Foo o) {
         return getName ().compareTo (o.getName ());
       }
 CODE;
   }
&lt;}
</pre>


#### Imports

Specifies one or more import directives to be placed at the top of the `.java` file. 
This is necessary if classes are referenced in a `body` section of code. 
It takes a comma separated list of classes (or packages with the wildcard import). 
For example:

<pre>
 message Foo {
   required int n;
   binding Java {
     imports "java.io.InputStream, java.io.IOException";
     body &lt;&lt;&lt;CODE
       public static Foo createFromStream (InputStream is) throws IOException {
         ... method body
       }
 CODE;
   }
 }
</pre>


#### Methods

Overrides the settings for message methods `equals`, `hashCode` and `toString`. 
It is a comma separated list of directives, for example:

<pre>
 message Foo {
   binding Java {
     methods "no-equals, toString";
   }
 }
</pre>

The directives available are:

| Directive | Meaning |
| equals | Generate equals method for this message |
| no-equals | Do not generate equals method for this message |
| hashCode | Generate hashCode method for this message |
| no-hashCode | Do not generate hashCode method for this message |
| toString | Generate toString method for this message |
| no-toString | Do not generate toString method for this message |


### Code generator patterns

For a field `foo` within the message, the following elements may be included within the class:

| Element | Name | Notes |
| field | _foo | Private internal value of the field. If not declared as mutable in the `.proto` file, will also be final. |
| mutator | setFoo(foo) | For mutable fields only. |
| accessor | getFoo() | Returns the value of the field. If the field is repeated, returns an unmodifiable list. |

If there are immutable, but optional fields (e.g. a default value), a builder pattern will be used.
 Otherwise a public constructor will be created which will take each of the `required` fields in the 
 order they are listed in the `.proto` file.

When the builder pattern is used, an inner `Builder` class is defined with a constructor that takes each 
of the `required` fields in the order they are listed into the `.proto` file. Any optional fields can 
be specified by calling a method on the builder.

For example:

<pre>
 /* Foo.proto */
 message Foo {
   required string name;
   required string email;
 }
</pre>

Will not require the builder pattern and create the following (some items omitted for clarity):

<pre>
 /* Foo.java */
 public class Foo {
   private final String _name;
   private final String _email;
   public Foo (final String name, final String email) {
     _name = name;
     _email = email;
   }
   public String getName () {
     return _name;
   }
   public String getEmail () {
     return _name;
   }
 }
</pre>

Whereas, if a fields is made optional the builder pattern will be used - for example:

<pre>
 /* Foo.proto */
 message Foo {
   required string name;
   optional string email;
 }
</pre>

Will generate (some artifacts have been omitted for clarity):

<pre>
 /* Foo.java */
public class Foo {
  public static class Builder {
    private final String _name;
    private String _email;
    public Builder (final String name) {
      _name = name;
    }
    public Builder email (final String value) {
      _email = value;
      return this;
    }
    public Foo build () {
      return new Foo (this);
    }
  }
  private final String _name;
  private final String _email;
  protected Foo (final Builder builder) {
    _name = builder._name;
    _email = builder._email;
  }
  public String getName () {
    return _name;
  }
  public String getEmail () {
    return _email;
  }
}
</pre>

Construction of a `Foo` object using the builder could look like:

<pre>
 final Foo.Builder fooBuilder = new Foo.Builder ("my name");
 fooBuilder.email ("my e-mail address");
 final Foo foo = fooBuilder.build ();
</pre>

Note that the methods on the `Builder` class to set optional values return a reference to the builder 
so that they can be chained in a manner similar to the `java.lang.StringBuilder` class.

Additional public elements in the classes may include:

| Element | Notes |
| equals method | Only if requested |
| hashCode method | Only if requested |
| toString method | Only if requested |
| copy constructor | Only public if the message has mutable fields so that safe assignments can be made |
| toFudgeMsg method | Converts the object to a FudgeFieldContainer |
| fromFudgeMsg static method | Constructs an object from a FudgeFieldContainer |

Further protected elements may exist to support the inheritance mechanism. These should not be called directly.


### External classes

An externally defined enumeration or message can be defined using the `extern` keyword. 
For example, to declare classes in package `com.mycompany.utils` as external:

<pre>
 namespace com.mycompany.utils {
   extern message Foo;
   extern enum Bar;
 }

 message Example {
   required com.mycompany.utils.Foo foo;
   required com.mycompany.utils.Bar bar;
 }
</pre>

External messages must implement a couple of methods to work correctly with the Fudge Proto generated code.


#### Enumerations

An enumeration should be declared as Java's `enum` type. It will be string encoded in a Fudge field 
using the value returned by the `.name()` method. An alternative declaration of an enumeration is any 
Java class that implements a `public Object name()` method that returns the value to be placed in a Fudge field.

An encoded field will be returned to a Java object using the deserialization framework. 
The default decoding behaviour is to call `Enum.valueOf` with the class and string value if can be 
converted to a primitive string, or if it is a sub-message use the string value of a field with 
ordinal 1 to support the form generated by the serialization framework. If the implementing class is 
not a Java `enum` it must be registered with the TypeDictionary as a secondary type to define the 
decoding strategy from the encoded type returned by its `name` method.


#### Messages

An external message must be implemented as a class or interface. Conversion to/from Fudge messages 
will be implemented through the Fudge serialization framework when messages are used as field types. 
Refer to the documentation for the `org.fudgemsg.mapping` package for full details.

If an external message will be used as the base for inheritance it must implement a method for serialization:

<pre>
 protected void toFudgeMsg (FudgeSerializationContext fudgeContext, MutableFudgeFieldContainer message) {
   ... adds any fields from this object to the message
 }
</pre>

It must also implement constructors for copying and for deserialization:

<pre>
 protected Foo (Foo source) {
   ... copy any data from source to this object
 }
 protected Foo (FudgeDeserializationContext fudgeContext, FudgeFieldContainer message) {
   ... construct this object from the message
 }
</pre>

It is likely that base classes will not have these methods if they are salvaged from other projects or 
part of a supplied package. For example:

<pre>
 /* OriginalBaseClass.proto */
 class OriginalBaseClass {
   private int _foo;
   public int getFoo () {
     return _foo;
   }
   public void setFoo (int foo) {
     _foo = foo;
   }
 }
</pre>

Instead of inheriting directly from the above class, creating an intermediate one which implements 
the required methods and uses the methods and constructor available in the super-class. For example:

<pre>
 /* IntermediateClass.proto */
 class IntermediateClass extends OriginalBaseClass {
   protected IntermediateClass (IntermediateClass source) {
     super ();
     setFoo (source.getFoo ());
   }
   protected IntermediateClass (FudgeDeserializationContext fudgeContext, FudgeFieldContainer message) {
     super ();
     setFoo (source.getInt ("foo"));
   }
   protected void toFudgeMsg (FudgeSerializationContext fudgeContext, MutableFudgeFieldContainer message) {
     message.add ("foo", getFoo ());
   }
 }
</pre>

Which then allows:

<pre>
 /* ExtendsBaseClass.proto */
 message ExtendsBaseClass extends IntermediateClass {
   required int bar;
 }
</pre>

Note that in this example the serialization contexts are not used but always passed into the constructor 
and `toFudgeMsg` methods, but may not be used if the full serialization framework is not needed to 
encode or decode a Fudge message. This is because the `.proto` compiler does not currently know enough 
of the external classes at code generation time to identify the most efficient call. 
A future version may address this and call versions which just take the message parameter if available.

