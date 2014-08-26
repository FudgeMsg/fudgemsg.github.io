---
layout: doc
title: Fudge-Proto command line
---

There are three basic ways to run the command line compiler:

* With antlr3 and the Fudge-Java implementation jar in the {{ext}} folder, the Fudge-Proto command line compiler can be launched as {{java \-jar fudge-proto.jar}} {{{_}options{_}}}.
* With antlr3, Fudge-Java and Fudge-Proto jars on the classpath, the command line compiler can be launched as {{java org.fudgemsg.proto.CommandLine}} {{{_}options{_}}}.
* If installed from the RPM bundle, it can be invoked as {{fudge-proto}} {{{_}options{_}{}}}_._

Available options (always starting with a - character) are:

| Option | Description | Example |
| -d*path* | Output folder for generated code. Will default to the same as the source folder. | -dproto_out |
| -f{_}modifier_ | Specify default modifiers for fields (normally fields are optional and mutable by default). | -freadonly -frequired |
| -l{_}language_ | Specify target language | -lJava |
| -s{_}path_ | Source folder for finding referenced files. Will default to the current folder. When an identifier is encountered that is not recognised, this folder is searched for a corresponding source file by turning the namespace delimiters into filesystem delimiters and appending ``.proto`` to the name. Files found in this manner will be compiled and code generated as if they were passed on the command line. | -ssrc |
| -p{_}path_ | Source folder for finding referenced files from another project. When an identifier is encountered that is not recognised, this folder is searched for a corresponding source file by turning the namespace delimiters into filesystem delimiters and appending ``.proto`` to the name. Files will be read but no code will be generated. This can be specified multiple times. | -p/projects/common/src -p/projects/other/src |
| -X{_}flag_ | Custom flag for the code generator selected by `-l`. | -Xcomments |
| -X{_}option_=_value_ | Custom option for the code generator selected by `-l`. | -Xstyle=compact |

Anything not matching is treated as a filename to be processed.
Wild cards can only be used if the operating system shell supports them and performs the expansion to discreet filenames.

Any errors or warnings will be written to `stderr`.
If one or more errors occurs the tool will exit with a non-zero error code.
If no errors occur the error code will be zero.

### Further information

* [Java command line options](fudge-proto-java.html)
