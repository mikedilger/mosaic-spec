# Mosaic Core Tags

<status>PAGE STATUS: early draft</status>

Tags are laid out as follows:

```text
0           2      3         256 max
+-----------+------+----------+
| type      | len  | value ...|
+-----------+------+----------+
```

* `[0:2]` - The type
* `[2:3]` - The length of the entire tag including the 3 byte header
    This must be at least 3.
* `[3:]` - The value, which is at most 253 bytes long.

Rationale

* While for some tag types a length could be inferred, this is not
  true in general. Applications are not required to recognize every
  tag type to look up its known length or method of length
  calculation.

Some tag types start with padding in the value in order to better align
their data.

## Notify Public Key

> **0x1**

```text
      0       2    3                  8
 0x0  +-------------------------------+ 0
      |  0x1  |0x28|       0x0        |
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

* `[0:2]` - The type 0x1 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x28
* `[3:8]` - Zeroed
* `[8:40]` - public key (32 bytes)

Records with this tag indicate that the record is if interest to the
person identified by that public key (as their master key).  Being tagged
as such, it should be delivered to all of this persons' INBOX servers as
specified in their [bootstrap](bootstrap.md) record.

## Reply by Hash

> **0x2**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x2  |0x28| 0|     KIND      |
 0x8  +-------------------------------+ 8
      | HASH 1/4                      |
 0x10 +-------------------------------+ 16
      | HASH 2/4                      |
 0x18 +-------------------------------+ 24
      | HASH 3/4                      |
 0x20 +-------------------------------+ 32
      | HASH 4/4                      |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The type 0x2 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x28
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md)
* `[8:40]` - The hash (32 bytes)

This is a reply to another record in a threading sense.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one replies to. Replies are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`HASH` is a 32-byte hash [reference](reference.md) to some other record
indicating which other record this record replies to.

If a record includes this tag, it must also include a
[Root by Addr](#root-by-addr) tag as well.

## Reply by Addr

> **0x3**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x3  |0x38| 0|     KIND      |
 0x8  +-------------------------------+ 8
      | ADDR 1/6                      |
 0x10 +-------------------------------+ 16
      | ADDR 2/6                      |
 0x18 +-------------------------------+ 24
      | ADDR 3/6                      |
 0x20 +-------------------------------+ 32
      | ADDR 4/6                      |
 0x28 +-------------------------------+ 40
      | ADDR 5/6                      |
 0x30 +-------------------------------+ 48
      | ADDR 6/6                      |
 0x38 +-------------------------------+ 56
```

* `[0:2]` - The type 0x3 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x38
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md)
* `[8:56]` - The address (48 bytes)

This is a reply to another record in a threading sense.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one replies to. Replies are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`ADDR` is a 48-byte address [reference](reference.md) to some other
record indicating which other record this record replies to.

If a record includes this tag, it must also include a
[Root by Addr](#root-by-addr) tag as well.

## Root by Hash

> **0x4**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x4  |0x28| 0|     KIND      |
 0x8  +-------------------------------+ 8
      | HASH 1/4                      |
 0x10 +-------------------------------+ 16
      | HASH 2/4                      |
 0x18 +-------------------------------+ 24
      | HASH 3/4                      |
 0x20 +-------------------------------+ 32
      | HASH 4/4                      |
 0x28 +-------------------------------+ 40
```

* `[0:2]` - The type 0x4 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x28
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md)
* `[8:40]` - The hash (32 bytes)

This indicates the root of the reply thread. This is to support loading
an entire thread in one round trip.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that the root record is. Threads are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`HASH` is a 32-byte hash [reference](reference.md) to some other record
indicating which other record this record replies to.

If a record includes this tag, it must also include a
[Reply by Hash](#reply-by-hash) or [Reply by Addr](#reply-by-addr) tag
as well.

## Root by Addr

> **0x5**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x5  |0x38| 0|     KIND      |
 0x8  +-------------------------------+ 8
      | ADDR 1/4                      |
 0x10 +-------------------------------+ 16
      | ADDR 2/4                      |
 0x18 +-------------------------------+ 24
      | ADDR 3/4                      |
 0x20 +-------------------------------+ 32
      | ADDR 4/4                      |
 0x28 +-------------------------------+ 40
      | ADDR 5/4                      |
 0x30 +-------------------------------+ 48
      | ADDR 6/4                      |
 0x38 +-------------------------------+ 56
```

* `[0:2]` - The type 0x5 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x38
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md)
* `[8:56]` - The address (48 bytes)

This indicates the root of the reply thread. This is to support loading
an entire thread in one round trip.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that the root record is. Threads are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`ADDR` is a 48-byte address [reference](reference.md) to some other
record which is the root of the thread.

If a record includes this tag, it must also include a
[Reply by Hash](#reply-by-hash) or [Reply by Addr](#reply-by-addr) tag
as well.

## Content Segment: User Mention

> **0x20

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  0x20 |0x28|0x0|   OFFSET     |
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

* `[0:2]` - The type 0x20 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x28
* `[3:4]` - Zeroed
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:40]` - public key (32 bytes)

This is a mention of a person.

`OFFSET` is the offset into the content where the mention appears.

`PUBLIC_KEY` is the master public key of the person mentioned.

Note that this is different from a [Notify Public Key](#notify-public-key) tag
which indicates the event should be delivered to that person.  Instead, this
tag indicates that a `@name` for the person should be rendered when rendering
the content.

## Content Segment: Server Mention

> **0x21**

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  0x21 |0x28|0x0|   OFFSET     |
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

* `[0:2]` - The type 0x20 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x28
* `[3:4]` - Zeroed
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:40]` - public key (32 bytes)

This is a mention of a server.

`OFFSET` is the offset into the content where the mention appears.

`PUBLIC_KEY` is the public key of the server.

## Content Segment: Quote by Id

> **0x22**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x22 |0x30| 0|     KIND      |
 0x8  +-------------------------------+ 8
      |     0x0       |    OFFSET     |
 0x10 +-------------------------------+ 16
      | HASH 1/4                      |
 0x18 +-------------------------------+ 24
      | HASH 2/4                      |
 0x20 +-------------------------------+ 32
      | HASH 3/4                      |
 0x28 +-------------------------------+ 40
      | HASH 4/4                      |
 0x30 +-------------------------------+ 48
```

* `[0:2]` - The type 0x22 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x30
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md) of the quoted record
* `[8:12]` - Zeroed
* `[12:16]` - The offset as a little-endian encoded unsigned integer.
* `[16:48]` - The hash (32 bytes) of the quoted record

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one replies to. Replies are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`OFFSET` is the offset into the content where the mention appears.

`HASH` is the hash of the quoted record.

This is a quote of another record by id.

## Content Segment: Quote by Addr

> **0x23**

```text
      0       2   3   4               8
 0x0  +-------------------------------+ 0
      |  0x23 |0x40| 0|     KIND      |
 0x8  +-------------------------------+ 8
      |     0x0       |    OFFSET     |
 0x10 +-------------------------------+ 16
      | ADDR 1/4                      |
 0x18 +-------------------------------+ 24
      | ADDR 2/4                      |
 0x20 +-------------------------------+ 32
      | ADDR 3/4                      |
 0x28 +-------------------------------+ 40
      | ADDR 4/4                      |
 0x30 +-------------------------------+ 48
      | ADDR 5/6                      |
 0x38 +-------------------------------+ 56
      | ADDR 6/6                      |
 0x40 +-------------------------------+ 64
```

* `[0:2]` - The type 0x23 as a little-endian encoded unsigned integer
* `[2:3]` - The length 0x40
* `[3:4]` - Zeroed
* `[4:8]` - The [kind](kinds.md) of the quoted record
* `[8:12]` - Zeroed
* `[12:16]` - The offset as a little-endian encoded unsigned integer.
* `[16:64]` - The address (48 bytes) of the quoted record

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one replies to. Replies are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`OFFSET` is the offset into the content where the mention appears.

`HASH` is the hash of the quoted record.

This is a quote of another record by address.

## Content Segment: URL

> **0x24**

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  0x24 |LEN |0x0|   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The type 0x24 as a little-endian encoded unsigned integer
* `[2:3]` - The length of the tag (8 + the length of the URL)
* `[3:4]` - Zeroed
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:]` - The URL to be included (up to 248 bytes long)

`OFFSET` is the offset into the content where the mention appears.

`URL` is the URL to be inserted at the offset.

This is a URL to a web page.

## Content Segment: Image

> **0x25**

```text
      0       2    3   4              8
 0x0  +-------------------------------+ 0
      |  0x25 |LEN |0x0|   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The type 0x25 as a little-endian encoded unsigned integer
* `[2:3]` - The length of the tag (8 + the length of the URL)
* `[3:4]` - Zeroed
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
      |  0x26 |LEN |0x0|   OFFSET     |
 0x8  +-------------------------------+ 8
      | URL...                        |
      +-------------------------------+
```

* `[0:2]` - The type 0x26 as a little-endian encoded unsigned integer
* `[2:3]` - The length of the tag (8 + the length of the URL)
* `[3:4]` - Zeroed
* `[4:8]` - The offset as a little-endian encoded unsigned integer.
* `[8:]` - The URL to be included (up to 248 bytes long)

`OFFSET` is the offset into the content where the mention appears.

`URL` is the URL to be inserted at the offset.

This is a URL to a video
