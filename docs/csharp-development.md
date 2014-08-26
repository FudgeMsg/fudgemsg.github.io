---
layout: doc
title: C# development
---

* The [GitHub home of the C# Reference Implementation](https://github.com/FudgeMsg/Fudge-CSharp)


### CSharp release process

Basic steps for doing a release build:

* Update the version in the AssemblyInfo.cs files for both the Fudge and FudgeTests projects.
* Build in Release configuration.
* Grab the dlls from the bin/Release directories.
* Go to the pub

Building the API reference:

* Install the Sandcastle Help File Builder.
* Open Fudge.shfbproj.
* Build it - this will create a Help folder containing the output .chm file.


### Fudge-CSharp 0.1

Key feature summary:

* Implementation of core FudgeMsg and primitive types.
* Support for "secondary" types which encode as a primitive type.
* Linq support.

API documentation is available as a help file
[here](http://fudgemsg.org/download/attachments/2359301/FudgeDocs.zip?version=2&modificationDate=1258795442241).


### Getting started

This guide is intended for people wishing to develop Fudge-CSharp itself.

#### What you'll need

At a bare minimum, you'll need:

* An account on [GitHub|http://github.com/] and something like [GitGui](http://code.google.com/p/msysgit/downloads/list) to access it.
* [Visual C# 2008 Express](http://www.microsoft.com/express/vcsharp/) or better.
* [XUnit](http://www.codeplex.com/xunit) (used for unit tests)

#### Running unit tests manually

Ideally you'll be using something like [TestDriven.net](http://www.testdriven.net/quickstart.aspx) to run
your tests from within Visual Studio, but if you're in the Express edition then plug-ins
are disabled (for a discussion on why, have a look at
[this article](http://blogs.msdn.com/danielfe/archive/2007/05/31/visual-studio-express-and-testdriven-net.aspx)).


If you need to run the tests manually from the command-line, then make sure your XUnit installation directory
is on your path, and then from your build output directory (e.g. `FudgeTests\bin\Release`) run the following:

<pre>
 xunit.console.exe FudgeTests.dll
</pre>

This will generate your usual console-style output:

<pre>
 xUnit.net console test runner (64-bit .NET 2.0.50727.4200)
 Copyright (C) 2007-9 Microsoft Corporation.

 xunit.dll: Version 1.4.9.1465
 Test assembly: c:\Projects\Fudge\Fudge-CSharp\FudgeTests\bin\Release\FudgeTests.dll

 ..........................................................
 Total tests: 58, Failures: 0, Skipped: 0, Time: 1.865 seconds
</pre>

#### TODO

* Assembly structure
* Coding conventions, inc. commenting style and header block
* Unit testing guidelines

