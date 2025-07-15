# Kind

The kind of a record determines the application that the record is used by.
This also determines the nature of the payload.

Kinds are represented by an array of 8 bytes, equivalent to a 64-bit unsigned
integer in big endian format.

A kind can be broken into three parts:

|Range|Meaning|
|-----|--------|
|Byte 0-3|Application Identifier, 32-bit unsigned integer in big-endian format|
|Byte 4-5|Application-Specific Kind, 16-bit unsigned integer in big-endian format|
|Byte 6-7|Bitflags for record handling|

Bitflags are as follows:

Bits 1 and 0:

* 00 - `Unique`. All records SHOULD have unique addresses. In the case that multiple
       records share the same address, all of them MUST be preserved just like
       Versioned records are (see bits 11 below)
* 01 - `Ephemeral`; Servers MUST serve this record to current subscribers, but
       SHOULD NOT save the record nor serve it later to future subscribers.
* 10 - `Replaceable`: Among records with the same address, only the one with the
       latest timestamp MUST be served by servers, and prior records MUST NOT be
       served.
* 11 - `Versioned`: Among records with the same address, all of them remain relevant
       and should be seen as a version history.

Bits 3 and 2:

* 00 - `Author Only`: Servers MUST serve this record only to it's author.
* 01 - `Author + Tagged`: Servers MUST serve this record only to it's author and the
       public keys that are tagged in the record.
* 10 - RESERVED
* 11 - `Everybody`: Servers MAY serve this record to anybody.

Bit 4: If on, the contents of this record are considered to be `printable` (human readable).

Bits 15-5: RESERVED and MUST be 0
