---
layout: doc
title: Date and Time Type
---

Three Standard [Types](types.html) are available for encoding dates and times:

* a date on its own
* a time on its own
* a combined date with a time

Fudge date/time encoding aims to support most language implementations by supporting optional timezone information,
and from year to nanosecond precision.

### Date

A date is encoded as a 4-byte value containing a year, month, and day packed as:

| Bits   | Meaning |
| 31 - 9 | Year (signed 2s complement to allow negative values for BCE/BC)|
| 8 - 5  | Month of year (1..12, or 0 to omit)|
| 4 - 0  | Day of month (1..31, or 0 to omit)|

The month and day, or just the day can be omitted by using the value 0. Values of 13 or higher are not currently
valid in the month field. For example:

| Date            | Coded values   | Packed value |
| 31 January 2010 | 2010, 1, 31    | `0000 0000 0000 1111 1011 010  0001  11111` |
| August 2000     | 2000, 8, 0     | `0000 0000 0000 1111 1010 000  1000  00000` |
| 3,000,000 BC    | -3000000, 0, 0 | `1010 0100 0111 0010 1000 000  0000  00000` |

Note the colors are to show where the fields start/end in the packed value - they have no other meaning.
Clearly some values are not valid dates and should not be used. The behavior of an invalid date value is undefined.
A system may either ignore any attempt to validate it and pass the value unchanged within a message, or if
there is validation as part of conversion to a native language type trigger an appropriate error.

Note that year zero is not valid, as year=1 is used for 1CE/AD and year=-1 for 1BCE/BC.

From 2013-03-13, the specification has been extended to support two special-case values, for minimum/far-past
and maximum/far-future. These can be mapped to concepts in a date library, such as JSR-310 `LocalDate.MAX`.

| Date              | Packed value                                                                       |
| MAX or far-future | `0111 1111 1111 1111 1111 111  1111  11111` |
| MIN or far-past   | `1000 0000 0000 0000 0000 000  1111  11111` |

This corresponds to using month=15 and day=31 as a marker to indicate max/min, with the year set to the largest
or smallest possible. This design choice was made to be as backwards compatible as reasonably
possible with older readers.


### Time

A time (time of day) is encoded as 8 bytes, holding data as follows:

|Bits     |Meaning          |Encoding |
| 63 - 56 | Timezone offset | Offset from UTC in 15-minute intervals. Signed 8-bit integer. Special value -128 means no timezone information |
| 55 - 52 | Accuracy        | Enumerated value - see below |
| 51 - 49 | Unused          | Set to zero |
| 48 - 32 | Seconds since midnight | Unsigned 17-bit integer* |
| 31 - 29 | Unused          | Set to zero |
| 29 - 0  | Nanoseconds since the start of the second | Unsigned 30-bit integer* |

* The valid ranges for seconds since midnight and nanoseconds since start of the second are encoded as unsigned values.
They are aligned to 32 bit boundaries so an implementation may set the most significant bits to 0 and work
with 32-bit signed integers internally if preferred.

Timezone examples:

| GMT  | Greenwich Mean Time   | London    | UTC+0H | 0 |
| CET  | Central European Time | Frankfurt | UTC+1H | 4 |
| PST  | Pacific Standard Time | LA        | UTC-8H | -32 |
| ACST | Australian Central Standard Time | Darwin | UTC+9:30H | 38 |

If further details of the Timezone are required, beyond the offset, they cannot be represented within this type.
Any such information is likely to be specific to an application's requirement for processing date/time information
and so should be sent as one or more separate fields in an application defined manner.

Accuracy is an enumeration (ordered to follow the English semantics of "greater" precision meaning smaller):

* 0 = Millennium
* 1 = Century
* 2 = Year
* 3 = Month
* 4 = Day
* 5 = Hour
* 6 = Minute
* 7 = Second
* 8 = Millisecond
* 9 = Microsecond
* 10 = Nanosecond

Values of 0 through 4 are only valid for DateTime (see below).


### Combined date and time

The combined date and time value is a 12-byte sequence -
the 4-byte date as encoded above, followed by the 8-byte time as encoded above.
