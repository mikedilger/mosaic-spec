# Filter

<status>PAGE STATUS: early draft</status>

A <t>filter</t> is a binary structure used within the [Core Protocol](protocol.md).

A <t>filter</t> consists of a sequence of <t>filter element</t>s.

Each <t>filter element</t> restricts the set of records that the <t>filter</t> matches.
For a record to pass a <t>filter</t>, it must pass every <t>filter element</t> in the
<t>filter</t> except as excepted in the next paragraph.

All <t>filter element</t>s SHOULD be unique. If more than one <t>filter element</t> of
the same type exists, only the first one counts. Subsequent ones MUST be ignored.

Some <t>filter element</t>s are narrow, meaning they select just a few records among
many. Other <t>filter element</t>s are wide and select many or even most records.
<T>Filter</T>s SHOULD include at least one narrow <t>filter element</t> in order
to narrow down the set of matching records to something reasonable. Software
MAY reject <t>filter</t>s that do not comply with this requirement.
Narrow <t>filter element</t>s have a type number that has the top 4 bits clear (less than 0x80).

Each <t>filter element</t> contains it's type in the first byte, and a length byte as
it's second byte as a count of the number of 8-byte words, so multiply
this by 8 to get the length of the <t>filter element</t> in bytes. This means that
<t>filter element</t>s can be up to 2048 bytes long. If the length byte is 0, the
<t>filter</t> is invalid and MUST be rejected.

<T>Filter</T>s can be up to 65536 bytes long maximum, but this size may not be quite
possible given other constraints.


## Filter Structure

It is defined as a simple header followed by a contiguous sequence of <t>filter element</t>s.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |bytelen|          0x0          |
 8  +-------------------------------+
    | Filter Elements ...           |
    +-------------------------------+
```

* `[0:2]` - The full byte length of the <t>filter</t> in little-endian. This MUST be
             divisible by 8 since all filter elements are multiple of 8 in length.
* `[2:8]` - Zeroed
* `[8:]` - The contiguous sequence of <t>filter element</t>s




<hr>

The following <t>filter element</t>s are defined:

|type|name|narrow|
|----|----|------|
|0x1|[Author Keys](#author-keys)| yes |
|0x2|[Signing Keys](#signing-keys)| yes |
|0x3|[Kinds](#kinds)| yes |
|0x4|[Timestamps](#timestamps)| yes |
|0x5|[Includes Tag](#includes-tag)| yes |
|0x80|[Since](#since)| no |
|0x81|[Until](#until)| no |
|0x82|[Received Since](#received-since)| no |
|0x83|[Received Until](#received-until)| no |
|0x84|[Exclude](#exclude)| no |
|0x85|[Excludes Tag](#excludes-tag)| no |

## Author Keys

> **0x1**

Matches all records authored by any of these author keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x1|len|      0x0              |
 8  +-------------------------------+
    | AUTHOR KEY 1/4                |
 16 +-------------------------------+
    | AUTHOR KEY 2/4                |
 24 +-------------------------------+
    | AUTHOR KEY 3/4                |
 32 +-------------------------------+
    | AUTHOR KEY 4/4                |
 40 +-------------------------------+
    | ... additional keys..         |
    +-------------------------------+
```

* `[0:1]` - The type 0x1
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of 32-byte author public keys.


## Signing Keys

> **0x2**

Matches all records signed by any of these keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x2|len|         0x0           |
 8  +-------------------------------+
    | SIGNING KEY 1/4               |
 16 +-------------------------------+
    | SIGNING KEY 2/4               |
 24 +-------------------------------+
    | SIGNING KEY 3/4               |
 32 +-------------------------------+
    | SIGNING KEY 4/4               |
 40 +-------------------------------+
    | ... additional keys..         |
    +-------------------------------+
```

* `[0:1]` - The type 0x2
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of 32-byte signing public keys.


## Kinds

> **0x3**

Matches all records which are of any one of these kinds.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x3|len|        0x0            |
 8  +-------------------------------+
    |  KIND ...                     |
 16 +-------------------------------+
    |  ... additional kinds ...     |
    +-------------------------------+
```

* `[0:1]` - The type 0x3
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words.
* `[2:8]` - Zeroed
* `[8:]` - A sequence of `len-1` 8-byte [kinds](kinds.md) in little-endian format.

## Timestamps

> **0x4**

Matches all records that have any of these exact timestamps.
Typically used as part of address lookups.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x4|len|        0x0            |
 8  +-------------------------------+
    |             TIMESTAMP         |
 16 +-------------------------------+
    |             TIMESTAMP         |
 24 +-------------------------------+
    | ...                           |
```

* `[0:1]` - The type 0x4
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of 8-byte fields, each being:
    * `[0:8]` - An eight byte [timestamp](timestamps.md).

## Includes Tag

> **0x5**

Matches all records that contain any of the given tags.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x5|len|          0x0          |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0x5
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of [Tags](record.md#tags), each starting with a 2-byte type
           and a 1 byte data length, then up to 253 bytes of data.
           This MUST be followed by padding that is all zeroes if the end
           of the last tag does not hit a 64-bit boundary. When parsing,
           reject anything less than 3 bytes, and if a parsed tag has
           type 0x0 it is padding rather than a tag.



## Since

> **0x80**

Matches all records with a timestamp greater than or equal to
this value. [<sup>rat</sup>](rationale.md#timestamp-ranges)

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x80|len|         0x0          |
 8  +-------------------------------+
    |         TIMESTAMP             |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x80
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:16]` - An eight byte [timestamp](timestamps.md).

## Until

> **0x81**

Matches all records with a timestamp less than or equal to
this value. [<sup>rat</sup>](rationale.md#timestamp-ranges)

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x81|len|         0x0          |
 8  +-------------------------------+
    |          TIMESTAMP            |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x81
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:16]` - An eight byte [timestamp](timestamps.md).

## Received Since

> **0x82**

Matches all records that were received by the server at or later
than this value. [<sup>rat</sup>](rationale.md#timestamp-ranges)

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x82|len|         0x0          |
 8  +-------------------------------+
    |          TIMESTAMP            |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x82
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:16]` - An eight byte [timestamp](timestamps.md).

## Received Until

> **0x83**

Matches all records that were received by the server at or before
this value. [<sup>rat</sup>](rationale.md#timestamp-ranges)

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x83|len|         0x0          |
 8  +-------------------------------+
    |         TIMESTAMP             |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x83
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:16]` - An eight byte [timestamp](timestamps.md).


## Exclude

> **0x84**

Excludes all records with the given IDs

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x84|len|        0x0           |
 8  +-------------------------------+
    |    ID prefix bytes 0..8       |
16  +-------------------------------+
    |    ID prefix bytes 8..16      |
24  +-------------------------------+
    |    ID prefix bytes 16..24     |
32  +-------------------------------+
    |    ID prefix bytes 24..32     |
40  +-------------------------------+
    | ... additional references     |
    +-------------------------------+
```

* `[0:1]` - The type 0x84
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of 32-byte ID prefixes.

## Excludes Tag

> **0x85**

Matches all records that do NOT contain any of the given tags.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x85|len|          0x0         |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0x85
* `[1:2]` - The length of the <t>filter element</t> in 8-byte words
* `[2:8]` - Zeroed
* `[8:]` - A sequence of [Tags](record.md#tags), each starting with a 2-byte type
           and a 1 byte data length, then up to 253 bytes of data.
           This MUST be followed by padding that is all zeroes if the end
           of the last tag does not hit a 64-bit boundary. When parsing,
           reject anything less than 3 bytes, and if a parsed tag has
           type 0x0 it is padding rather than a tag.
