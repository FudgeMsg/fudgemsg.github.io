---
layout: doc
title: Java development
---

The Java reference implementation provides Fudge encoding on the Java platform.

* The [GitHub home of the Java Reference Implementation](https://github.com/FudgeMsg/Fudge-Java)
* The [Java Downloads](https://github.com/FudgeMsg/Fudge-Java/releases)
* [Javadocs for the 0.3 version](http://dist.fudgemsg.org/java/javadoc/0.3/)
* [Javadocs for the 0.2 version](http://dist.fudgemsg.org/java/javadoc/0.2/)
* [Javadocs for the 0.1 version](http://dist.fudgemsg.org/java/javadoc/0.1/) 


### Development

This guide is for those who want work on the internals of the Fudge-Java reference implementation,
not for those who just wish to **use** the Fudge reference implementation.

#### Pre-Requisites
In order to work with the Java Reference Implementation, you will require the following components:

* **Java 6**. Come on, people, Java 5 is officially EOLed. If you're not on Java 6, you should be.
* [Ant](http://ant.apache.org/) 1.7

If you want to develop in Eclipse (and we maintain `.project` and `.classpath` files for Eclipse 3.5), you'll want to have as plugins:

* [IvyDE](http://ant.apache.org/ivy/ivyde/) as the Fudge Java dependencies are managed using Ivy
* [EGit](http://www.eclipse.org/egit/) if you want to access your local git repository

#### Pulling The Source
First of all, if you think you might want to do submissions, we recommend that you make a public fork
of the [Fudge-Java Github Repository](http://github.com/FudgeMsg/Fudge-Java).
While it's not strictly necessary, we much prefer pull requests in Github than patches emailed.

Then, just do a `git clone` of your Github URL and you've got the source.

#### Initial Build
Although in theory IvyDE is smart enough to do this itself, we've found that in general it's easiest
to run an Ant-based build first. Just run `ant` in your `Fudge-Java` directory and Ivy should pull down
all the dependencies (from the Fudge Dist site) and it'll build.

#### Eclipse Project Setup
Then, import a project in Eclipse and point it at your `Fudge-Java` directory, and you'll be up and running.
