# Kind

<status>PAGE STATUS: Early Draft</status>

The kind of a record determines the application that the record is used by.
This also determines the nature of the payload.

Kinds are represented by an 8-byte array of bits, not interpreted as an
integer.

A kind can be broken into three parts:

|Range|Meaning|
|-----|--------|
|Byte 0-4|Application Identifier|
|Byte 5-6|Differentiation within the Application|
|Byte 7|Bitflags for record handling|

Bitflags are as follows:

Bits 1 and 0:

* 00 - `Unique`. All records SHOULD have unique addresses. In the case that multiple
       records share the same address, all of them must be preserved just like
       Versioned records are (see bits 11 below)
* 01 - `Ephemeral`; Servers should serve this record to current subscribers, but
       should not save the record nor serve it later to future subscribers.
* 10 - `Replaceable`: Among records with the same address, only the one with the
       latest timestamp should be served by servers.
* 11 - `Versioned`: Among records with the same address, all of them remain relevant
       and should be seen as a version history.

Bits 3 and 2:

* 00 - `Author Only`: Servers MUST serve this record only to it's author.
* 01 - `Author + Tagged`: Servers MUST serve this record only to it's author and the
       public keys that are tagged in the record.
* 10 - RESERVED
* 11 - `Everybody`: Servers MAY serve this record to anybody.

Bit 4: If on, the contents of this record are considered to be `printable` (human readable).

Bits 7, 6 and 5: RESERVED and MUST be 0

## Application Registry

| Application |  ID   |
|-------------|-------|
| Mosaic Core |     0 |
| Mosaic Social Media |    1 |

External groups may define applications and have their Application ID registered here.

## Kind Registry

This only covers Mosaic Core and Mosaic Social Media. Other applications should define
their internal kinds within their own specifications, and simply register an Application
ID with Mosaic.


|App |App Kind|Name              |Flags             |Server Used|
|----|--------|------------------|------------------|-----------|
|Core| 1 |[Key Schedule](keyschedule.md)|Replace, All, NoPrint|Author Outbox|
|Core| 2 |[Profile](profile.md)|Replace, All, NoPrint|Author Outbox|
|Social Media| 1 |[Microblog Root](microblog.md)|Unique, All, Print |Author Outbox|
|Social Media| 2 |[Reply Comment](reply_comment.md)|Unique, All, Print |Author Outbox<br>Parent Author Inbox|
|Social Media| 3 |[Blog Post](blog.md)|Unique, All, Print |Author Outbox|
|Social Media| 4 |[Chat Message](chat.md)|Unique, All, Print |Chat Server|
