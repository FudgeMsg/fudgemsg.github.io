---
layout: doc
title: Indicator Type
---

`IndicatorType` is a special type that can be used to signal to consumers of Fudge encoded data that there is
a field present, but that the contents aren't meaningful in any way.
It is the only 0-sized fixed width type in the Fudge [type system](types.html).

It is commonly used for the following purposes:

* As a 0-byte boolean (if the field is there, `true`; else `false`)
* To clear a message when sending delta updates (previously field `foo` had the value 42; now, clear it from your current state)

In the reference implementations, there is a singleton instance of `IndicatorType` that is returned for all field references.
