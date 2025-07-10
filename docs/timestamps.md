# Timestamps

Timestamps are unixtimes, adjusted for leap seconds, expressed in
<t>nanoseconds</t> [<sup>rat</sup>](rationale.md#nanoseconds)
and encoded into a 64-bit unsigned integer in big-endian format.

The integer represents the number of nanoseconds that have actually elapsed
on the surface of the Earth since the UNIX epoch. The epoch is defined as (these
definitions are believed to be equivalent):

* 1 January 1970 UTC
* Unixtime 0
* NTP timestamp 2208988800
* Julian Date 2440587.5 UT1
* Julian Date 2440587.5004766666 TT
    * NOTE: the fracton above beyond .5 represents the 9 seconds that UTC was
      behind TAI, plus the 32.184 seconds that TAI is behind TT.

Timestamps account for all <t>leap seconds</t> [<sup>rat</sup>](rationale.md#leap-seconds)
unlike unixtime and unlike NTP timestamps (both of which pretend that leap seconds did not
happen).

Before 31 Dec 1971 23:59:60, timestamps match unixtime.

As of this writing (unixtime 1732829887) the current timetstamp which includes
28 additional leap seconds is 1732829915.  Expressed in nanoseconds it is
1_732_829_915_000_000_000. As a big-endian 64 bit unsigned integer its binary
representation is 0x180c_3fa0_73be_ce00.

Since computers tend to be synchronized with UTC for the time being, your
software will need to be aware of leap seconds so it can adjust.

Leap second data is available at the
[IANA leap second list](https://data.iana.org/time-zones/data/leap-seconds.list)
The data is to be interpreted as follows:

* The leftmost column is an NTP timestamp. Subtract 2_208_988_800 from it to
  get a unixtime.
* The rightmost column is an adjustment to TAI.  Subtract 9 from it to get the
  number of leapseconds that have elapsed as of the time in the first column and
  thereafter (until the next entry).
* A timestamp is the current unixtime, plus the number of leap seconds that have
  elapsed.

Refer to the [IANA leap second list](https://data.iana.org/time-zones/data/leap-seconds.list)
