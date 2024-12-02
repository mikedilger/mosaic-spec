# Timestamps

<status>PAGE STATUS: draft</status>

Timestamps are unixtimes, adjusted for leap seconds, expressed in milliseconds,
and encoded into a 48-bit unsigned integer in little-endian format.

The first bit is 0. Records that have a 1 bit here SHOULD be ignored.

The next 47 bits represent the number of milliseconds that have actually elapsed
on the surface of the Earth since the UNIX epoch. The epoch is defined as (these
definitions are believed to be equivalent):

* 1 January 1970 UTC
* Unixtime 0
* NTP timestamp 2208988800
* Julian Date 2440587.5 UT1
* Julian Date 2440587.5004766666 TT
    * NOTE: the fracton above beyond .5 represents the 9 seconds that UTC was
      behind TAI, plus the 32.184 seconds that TAI is behind TT.

Timestamps account for all leap seconds, unlike unixtime and unlike NTP timestamps
(both of which pretend that leap seconds did not happen).

Before 1 Jan 1972, timestamps match unixtime.

As of this writing (unixtime 1732829887) the current timetstamp which includes
28 additional leap seconds is 1732829915.

Since computers tend to be synchronized with UTC for the time being, your software
will need to be aware of leap seconds so it can adjust.

Leap second data is available at the [IANA leap second list](https://data.iana.org/time-zones/data/leap-seconds.list)
The data should be interpreted as follows:

* The leftmost column is an NTP timestamp. Subtract 2_208_988_800 from it to get a
  unixtime.
* The rightmost column is an adjustment to TAI.  Subtract 9 from it to get the
  number of leapseconds that have elapsed as of the time in the first column and
  thereafter (until the next entry).
* A timestamp is the current unixtime, plus the number of leap seconds that have
  elapsed.

RATIONALE:
* UTC is a discontinous time scale that is occasionally adjusted by leap seconds.
  Unixtime is derived from UTC and is thus also discontinuous.  Subtracting two
  unixtimes could give a time interval that is off by up to 28 seconds (for example
  when comparing dates before 1 Jan 1972 with today).
* The first bit is zero in case it is ever interpreted as a sign bit, in order
  to preserve sorting.
* Millisecond unixtimes in 47 bits give us more than 4000 years before they
  roll over.

Refer to the [IANA leap second list](https://data.iana.org/time-zones/data/leap-seconds.list)
