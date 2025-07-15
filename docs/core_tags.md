# Mosaic Core Tags

|Tag|
|---|
|[Notify Public Key](#notify-public-key)|
|[Reply](#reply)|
|[Root](#root)|
|[Nostr Sister Event](#nostr-sister-event)|
|[Subkey](#subkey)|
|[Content Segment: User Mention](#content-segment-user-mention)|
|[Content Segment: Server Mention](#content-segment-server-mention)|
|[Content Segment: Quote](#content-segment-quote)|
|[Content Segment: URL](#content-segment-url)|
|[Content Segment: Image](#content-segment-image)|
|[Content Segment: Video](#content-segment-video)|

## Notify Public Key

> **0x1**

```text
      0       2       4                  8
 0x0  +-------------------------------+ 0
      |  40   |  0x1  |   zeroed      |
 0x8  +-------------------------------+ 8
      | PUBLIC KEY 1/4                |
 0x10 +-------------------------------+ 16
      | PUBLIC KEY 2/4                |
 0x18 +-------------------------------+ 24
      | PUBLIC KEY 3/4                |
 0x20 +-------------------------------+ 32
      | PUBLIC KEY 4/4                |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The length 40 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x1 as a little-endian encoded unsigned integer
* `[4:8]` - Zeroed
* `[8:40]` - public key (32 bytes)

Records with this tag indicate that the record is of interest to the
person identified by that public key (as their master key).  Being tagged
as such, it SHOULD be delivered to all of this persons' INBOX servers as
specified in their [bootstrap](bootstrap.md) record.

## Reply

> **0x2**

```text
      0       2      4       6         8
 0x0  +-------------------------------+ 0
      |  64   | 0x2  |    zeroed      |
 0x8  +-------------------------------+ 8
      | KIND                          |
 0x10 +-------------------------------+ 16
      | REFERENCE 1/6                 |
 0x18 +-------------------------------+ 24
      | REFERENCE 2/6                 |
 0x20 +-------------------------------+ 32
      | REFERENCE 3/6                 |
 0x28 +-------------------------------+ 40
      | REFERENCE 4/6                 |
 0x30 +-------------------------------+ 48
      | REFERENCE 5/6                 |
 0x38 +-------------------------------+ 56
      | REFERENCE 6/6                 |
 0x40 +-------------------------------+ 64
```

* `[0:2]` - The length 64 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x2 as a little-endian encoded unsigned integer
* `[4:8]` - Zeroed
* `[8:16]` - The [kind](kind.md)
* `[16:64]` - The reference (48 bytes)

This is a reply to another record in a threading sense.

`KIND` is an 8-byte record [kind](kind.md) indicating the kind of record
that this one replies to. Replies are application-independent and MAY
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`REFERENCE` is a 48-byte [reference](reference.md) to some other record
indicating which other record this record replies to.

If a record includes this tag, it MUST also include a
[Root](#root) tag as well.

## Root

> **0x3**

```text
      0       2       4      6        8
 0x0  +-------------------------------+ 0
      |  64   |  0x3  |    zeroed     |
 0x8  +-------------------------------+ 8
      | KIND                          |
 0x10 +-------------------------------+ 16
      | REFERENCE 1/6                 |
 0x18 +-------------------------------+ 24
      | REFERENCE 2/6                 |
 0x20 +-------------------------------+ 32
      | REFERENCE 3/6                 |
 0x28 +-------------------------------+ 40
      | REFERENCE 4/6                 |
 0x30 +-------------------------------+ 48
      | REFERENCE 5/6                 |
 0x38 +-------------------------------+ 56
      | REFERENCE 6/6                 |
 0x40 +-------------------------------+ 64
```

* `[0:2]` - The length 64 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x3 as a little-endian encoded unsigned integer
* `[4:8]` - Zeroed
* `[8:16]` - The [kind](kind.md)
* `[16:64]` - The reference (48 bytes)

This indicates the root of the reply thread. This is to support loading
an entire thread in one round trip.

`KIND` is a 8-byte record [kind](kind.md) indicating the kind of record
that the root record is. Threads are application-independent and MAY
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`REFERENCE` is a 48-byte [reference](reference.md) to some other record
which is the root of the thread.

If a record includes this tag, it MUST also include a
[Reply](#reply) tag as well.

## Nostr Sister Event

> **0x8**

```text
      0       2       4               8
 0x0  +-------------------------------+ 0
      |  40   |  0x8  |    zeroed     |
 0x8  +-------------------------------+ 8
      | NOSTR ID 1/4                  |
 0x10 +-------------------------------+ 16
      | NOSTR ID 2/4                  |
 0x18 +-------------------------------+ 24
      | NOSTR ID 3/4                  |
 0x20 +-------------------------------+ 32
      | NOSTR ID 4/4                  |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The length 40 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x8 as a little-endian encoded unsigned integer
* `[4:8]` - Zeroed
* `[8:40]` - The Nostr ID (32 bytes)

For dual-stack clients that produce Nostr events alongside Mosaic records,
and who want to track replies on sister events in nostr as well as here in
Mosaic, this is a pointer to the sister event in nostr.

NOTE: The nostr sister event will have a "mosaic" tag that contains the
hex of the id of its Mosaic sister record.

## Subkey

> **0x10**

```text
      0       2        4              8
 0x0  +-------------------------------+ 0
      |  40   |  0x10  |    zeroed    |
 0x8  +-------------------------------+ 8
      | PUBLIC SUBKEY 1/4             |
 0x10 +-------------------------------+ 16
      | PUBLIC SUBKEY 2/4             |
 0x18 +-------------------------------+ 24
      | PUBLIC SUBKEY 3/4             |
 0x20 +-------------------------------+ 32
      | PUBLIC SUBKEY 4/4             |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The length 40 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x10 as a little-endian encoded unsigned integer
* `[4:8]` - Zeroed
* `[8:40]` - The subkey public key

This is used only on [key schedule](keyschedule.md) records so that clients
can look up a subkey to verify it's association to a master key.


## Content Segment: User Mention

> **0x20**

```text
      0       2        4              8
 0x0  +-------------------------------+ 0
      |  40   |  0x20  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | PUBLIC KEY 1/4                |
 0x10 +-------------------------------+ 16
      | PUBLIC KEY 2/4                |
 0x18 +-------------------------------+ 24
      | PUBLIC KEY 3/4                |
 0x20 +-------------------------------+ 32
      | PUBLIC KEY 4/4                |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The length 40 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x20 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:40]` - public key (32 bytes)

This is a mention of a person.

`OFFSET` is the offset into the content where the mention appears.

`PUBLIC_KEY` is the master public key of the person mentioned.

Note that this is different from a [Notify Public Key](#notify-public-key) tag
which indicates the record should be delivered to that person.  Instead, this
tag indicates that a `@name` for the person SHOULD be rendered when rendering
the content.

## Content Segment: Server Mention

> **0x21**

```text
      0       2        4              8
 0x0  +-------------------------------+ 0
      |  40   |  0x21  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | PUBLIC KEY 1/4                |
 0x10 +-------------------------------+ 16
      | PUBLIC KEY 2/4                |
 0x18 +-------------------------------+ 24
      | PUBLIC KEY 3/4                |
 0x20 +-------------------------------+ 32
      | PUBLIC KEY 4/4                |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The length 40 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x21 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:40]` - public key (32 bytes)

This is a mention of a server.

`OFFSET` is the offset into the content where the mention appears.

`PUBLIC_KEY` is the public key of the server.

## Content Segment: Quote

> **0x22**

```text
      0       2       4               8
 0x0  +-------------------------------+ 0
      |  64   |  0x22  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | KIND                          |
 0x10 +-------------------------------+ 16
      | REFERENCE 1/6                 |
 0x18 +-------------------------------+ 24
      | REFERENCE 2/6                 |
 0x20 +-------------------------------+ 32
      | REFERENCE 3/6                 |
 0x28 +-------------------------------+ 40
      | REFERENCE 4/6                 |
 0x30 +-------------------------------+ 48
      | REFERENCE 5/6                 |
 0x38 +-------------------------------+ 56
      | REFERENCE 6/6                 |
 0x40 +-------------------------------+ 64
```

* `[0:2]` - The length 64 as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x22 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:16]` - The [kind](kind.md) of the quoted record
* `[16:64]` - The id (48 bytes) of the quoted record

`OFFSET` is the offset into the content where the mention appears.

`KIND` is a 8-byte record [kind](kind.md) indicating the kind of record
that this one replies to. Replies are application-independent and MAY
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`REFERENCE` is a 48-byte [reference](reference.md) to the quoted record.

## Content Segment: URL

> **0x24**

```text
      0       2        4              8
 0x0  +-------------------------------+ 0
      |  LEN  |  0x24  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The length (8 plus the length of the URL) as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x24 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:]` - The URL to be included

`OFFSET` is the offset into the content where the mention appears.

`URL` is the URL to be inserted at the offset.

This is a URL to a web page.

## Content Segment: Image

> **0x25**

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  LEN  |  0x25  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The length (8 plus the length of the URL) as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x25 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:]` - The URL to be included (up to 248 bytes long)

`OFFSET` is the offset into the content where the mention appears.

`URL` is the URL to be inserted at the offset.

This is a URL to an image

## Content Segment: Video

> **0x26**

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  LEN  |  0x26  |   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The length (8 plus the length of the URL) as a little-endian encoded unsigned integer
* `[2:4]` - The type 0x26 as a little-endian encoded unsigned integer
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:]` - The URL to be included (up to 248 bytes long)

`OFFSET` is the offset into the content where the mention appears.

`URL` is the URL to be inserted at the offset.

This is a URL to a video
