# Filter

<status>PAGE STATUS: early draft</status>

A filter is a binary structure used within the [Core Protocol](protocol.md).

It is defined as a contiguous sequence of type-selector pairs.  Some selector
types specify a count or length within their value.

Each selector restricts the set of records that the filter matches. For an
event to pass the filter, it must pass all the selectors.

Selector types MUST not be used more than once within a filter.

Filters can be up to 65536 bytes long maximum, but this size may not be
possible given other constraints.

Filters SHOULD include at least one narrowly-tailored selector in order
to narrow down the set of matching events to something reasonable. Software
MAY reject filters that do not comply with this requirement.

The following selectors are defined:

|type|name|narrow|
|----|----|------|
|0x1|[Exclude](#exclude)| no |
|0x4|[Author Keys](#author-keys)| yes |
|0x5|[Signing Keys](#signing-keys)| yes |
|0x6|[Timestamps](#timestamps)| yes |
|0x7|[Since](#since)| no |
|0x8|[Until](#until)| no |
|0x9|[Received Ats](#received-ats)| yes |
|0xA|[Received Since](#received-since)| no |
|0xB|[Received Until](#received-until)| no |
|0xC|[Kinds](#kinds)| yes |
|0xD|[Includes Tag](#includes-tag)| yes |
|0xE|[Excludes Tag](#excludes-tag)| no |

## Exclude

> **0x1**

Excludes all records with the given IDs or Addresses.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x1|         0x0           | n |
 8  +-------------------------------+
    | ID or ADDR prefix bytes 0..8  |
16  +-------------------------------+
    | ID or ADDR prefix bytes 8..16 |
24  +-------------------------------+
    | ID or ADDR prefx bytes 16..24 |
32  +-------------------------------+
    | ID or ADDR prefx bytes 24..32 |
40  +-------------------------------+
    | ... additional references     |
    +-------------------------------+
```

* `[0:1]` - The type 0x1
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of references
* `[*]` - A sequence of `n` 32-byte ID or Address prefixes.

## Author Keys

> **0x4**

Matches all records authored by any of these keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x4|          0x0          | n |
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

* `[0:1]` - The type 0x4
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of author keys
* `[*]` - A sequence of `n` 32-byte author public keys.

## Signing Keys

> **0x5**

Matches all records signed by any of these keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x5|        0x0            | n |
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

* `[0:1]` - The type 0x5
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of signing keys
* `[*]` - A sequence of `n` 32-byte signing public keys.

## Timestamps

> **0x6**

Matches all records that have any of these exact timestamps.
Typically used as part of address lookups.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x6|        0x0            | n |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
    | ..0x0 |    ..TIMESTAMP        |
 24 +-------------------------------+
```

* `[0:1]` - The type 0x6
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of timestamp fields
* `[*]` - A sequence of `n` 8-byte fields, each being:
    * `[0:2]` - Zeroed
    * `[2:8]` - A six byte [timestamp](timestamps.md).

## Since

> **0x7**

Matches all records with a timestamp greater than or equal to
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x7|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x7
* `[1:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Until

> **0x8**

Matches all records with a timestamp less than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x8|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0x8
* `[1:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Received Ats

> **0x9**

Matches all records that were received at any of these exact
timestamps. This is unlikely to be useful but we add it for
completeness.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x9|          0x0          | n |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
    | ..0x0 |    ..TIMESTAMP        |
 24 +-------------------------------+
```

* `[0:1]` - The type 0x9
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of timestamp fields
* `[*]` - A sequence of `n` 8-byte fields, each being:
    * `[0:2]` - Zeroed
    * `[2:8]` - A six byte [timestamp](timestamps.md).

## Received Since

> **0xA**

Matches all records that were received by the server at or later
than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xA|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0xA
* `[1:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Received Until

> **0xB**

Matches all records that were received by the server before
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xB|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         |
 16 +-------------------------------+
```

* `[0:1]` - The type 0xB
* `[1:10]` - Zeroed
* `[10:16]` - A six byte [timestamp](timestamps.md).

## Kinds

> **0xC**

Matches all records which are of any one of these kinds.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xC|          0x0          | n |
 8  +-------------------------------+
    |   KIND        |  ...KIND      |
 16 +-------------------------------+
```

* `[0:1]` - The type 0xC
* `[1:7]` - Zeroed
* `[7:8]` - A 1-byte count `n` of kinds
* `[8]` - A sequence of `n` 4-byte [kinds](kinds.md)

## Includes Tag

> **0xD**

Matches all records that contain the given tag.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xD|0x0| TTYPE | 0x0   |  LEN  |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0xD
* `[1:2]` - Zeroed
* `[2:4]` - A 2-byte [tag type](tag_types.md) in little-endian format.
* `[4:6]` - Zeroed
* `[6:8]` - A 2-byte `LEN` unsigned integer in little-endian format
  indicating the length of the `VALUE`
* `[*]` - The `VALUE` of the tag, up to 253 bytes.

## Excludes Tag

> **0xE**

Matches all records that do NOT contain the given tag.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xE|0x0| TTYPE | 0x0   |  LEN  |
 8  +-------------------------------+
    |   VALUE...  .                 |
    +-------------------------------+
```

* `[0:1]` - The type 0xE
* `[1:2]` - Zeroed
* `[2:4]` - A 2-byte [tag type](tag_types.md) in little-endian format.
* `[4:6]` - Zeroed
* `[6:8]` - A 2-byte `LEN` unsigned integer in little-endian format
  indicating the length of the `VALUE`
* `[*]` - The `VALUE` of the tag, up to 253 bytes.
