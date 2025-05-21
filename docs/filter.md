# Filter

<status>PAGE STATUS: early draft</status>

A filter is a binary structure used within the [Core Protocol](protocol.md).

It is defined as a contiguous sequence of filter elements.

Each filter element restricts the set of records that the filter matches.
For a record to pass a filter, it must pass every filter element in the filter.

Some filter elements are narrow, meaning they select just a few records among
many. Other filter elements are wide and select many or even most records.
Filters SHOULD include at least one narrow filter element in order
to narrow down the set of matching events to something reasonable. Software
MAY reject filters that do not comply with this requirement.

Narrow filter elements have a type number that has the top 4 bits clear (less than 0x80).

Each filter element contains it's type in the first byte, and a length byte as
it's second byte. The length byte counts the number of 8-byte words, so multiply
by 8 to get the length of the filter element in bytes. This means that filter elements
can be up to 2048 bytes long. If the length byte is 0, the filter is invalid and MUST
be rejected.

Filters can be up to 65536 bytes long maximum, but this size may not be quite
possible given other constraints.

Most types of filter elements should only be used once, and provide for multiple
values. Tag filter elements can be used multiple times, once for each type of tag.

The following filter elements are defined:

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
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:8]` - Zeroed
* `[*]` - A sequence of `n` 32-byte author public keys.


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
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:8]` - Zeroed
* `[*]` - A sequence of `n` 32-byte signing public keys.


## Kinds

> **0x3**

Matches all records which are of any one of these kinds.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x3|len|        0x0            |
 8  +-------------------------------+
    |   KIND        |  ...KIND      |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x3
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:8]` - Zeroed
* `[8]` - A sequence of `n` 4-byte [kinds](kinds.md)

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
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
    | ..0x0 |    ..TIMESTAMP        |
 24 +-------------------------------+
```

* `[0:1]` - The type 0x4
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:8]` - Zeroed
* `[*]` - A sequence of `n` 8-byte fields, each being:
    * `[0:2]` - Zeroed
    * `[2:8]` - A six byte [timestamp](timestamps.md).

## Includes Tag

> **0x5**

Matches all records that contain the given tag.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x5|len| TTYPE |   0x0     |TL |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0x5
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:4]` - A 2-byte [tag type](tag_types.md) in little-endian format.
* `[4:7]` - Zeroed
* `[7:8]` - TL, a tag length in exact bytes, up to 253.
* `[*]` - The `VALUE` of the tag, of TL bytes in lenght, plus padding
          to align to 8 bytes.

## Since

> **0x80**

Matches all records with a timestamp greater than or equal to
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x80|len|         0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x80
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Until

> **0x81**

Matches all records with a timestamp less than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x81|len|         0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x81
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Received Since

> **0x82**

Matches all records that were received by the server at or later
than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x82|len|         0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x82
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Received Until

> **0x83**

Matches all records that were received by the server before
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x83|len|         0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x83
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).


## Exclude

> **0x84**

Excludes all records with the given references (IDs or Addresses).

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x84|len|        0x0           |
 8  +-------------------------------+
    | REFERENCE prefix bytes 0..8   |
16  +-------------------------------+
    | REFERENCE prefix bytes 8..16  |
24  +-------------------------------+
    | REFERENCE prefix bytes 16..24 |
32  +-------------------------------+
    | REFERENCE prefix bytes 24..32 |
40  +-------------------------------+
    | ... additional references     |
    +-------------------------------+
```

* `[0:1]` - The type 0x84
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:8]` - Zeroed
* `[*]` - A sequence of 32-byte reference prefixes.

## Excludes Tag

> **0x85**

Matches all records that do NOT contain the given tag.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x85|len|TTYPE | 0x0       |TL |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0x85
* `[1:2]` - The length of the filter element in 8-byte words
* `[2:4]` - A 2-byte [tag type](tag_types.md) in little-endian format.
* `[4:7]` - Zeroed
* `[7:8]` - TL, a tag length in exact bytes, up to 253.
* `[*]` - The `VALUE` of the tag, of TL bytes in length, plus padding
           to align to 8 bytes.
