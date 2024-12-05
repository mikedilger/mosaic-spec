# Filter

<status>PAGE STATUS: early draft</status>

A filter is a binary structure used within the [Core Protocol](protocol.md).

It is defined as a contiguous sequence of type-value pairs.  Some types
specify a count or length within their value.

Each entry restricts the set of records that the filter matches.

Filter types MUST not be used more than once within a filter.

Filters can be up to 65536 bytes long maximum, but this size may not be
possible given other constraints.

## Hash16

> **0x1**

Matches all records that have any of these hashes.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x1|          0x0          | n |
 8  +-------------------------------+
    | HASH PREFIX 1/2               | 
 16 +-------------------------------+
    | HASH PREFIX 2/2               |
 24 +-------------------------------+
    | ... additional hash prefixes..|
    +-------------------------------+
```

* The byte 0x1
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 16-byte, 128-bit hash prefixes.

## Hash32

> **0x2**

Matches all records that have any of these hashes.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x2|          0x0          | n |
 8  +-------------------------------+
    | HASH 1/4                      | 
 16 +-------------------------------+
    | HASH 2/4                      |
 24 +-------------------------------+
    | HASH 3/4                      |
 32 +-------------------------------+
    | HASH 4/4                      |
 40 +-------------------------------+
    | ... additional hash prefixes..|
    +-------------------------------+
```

* The byte 0x2
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 32-byte, 128-bit hash prefixes.

## Author Keys

> **0x3**

Matches all records authored by any of these keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x3|          0x0          | n |
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

* The byte 0x3
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 32-byte author public keys.

## Signing Keys

> **0x4**

Matches all records signed by any of these keys.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x4|        0x0            | n |
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

* The byte 0x4
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 32-byte signing public keys.

## Timestamps

> **0x5**

Matches all records that have any of these exact timestamps.
Typically used as part of address lookups.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x5|        0x0            | n |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
    | ..0x0 |    ..TIMESTAMP        |
 24 +-------------------------------+
```

* The byte 0x5
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 8-byte fields, each being:
    * Two bytes 0x0
    * A six byte [timestamp](timestamps.md).

## Since

> **0x6**

Matches all records with a timestamp greater than or equal to
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x6|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
```

* The byte 0x6
* Nine bytes 0x0
* A 6-byte timestamp


## Until

> **0x7**

Matches all records with a timestamp less than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x7|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
```

* The byte 0x7
* Nine bytes 0x0
* A 6-byte timestamp

## Received Ats

> **0x8**

Matches all records that were received at any of these exact
timestamps. This is unlikely to be useful but we add it for
completeness.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x8|          0x0          | n |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
    | ..0x0 |    ..TIMESTAMP        |
 24 +-------------------------------+
```

* The byte 0x8
* Six bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 8-byte fields, each being:
    * Two bytes 0x0
    * A six byte [timestamp](timestamps.md).

## Received Since

> **0x9**

Matches all records that were received by the server at or later
than this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0x9|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
```

* The byte 0x9
* Nine bytes 0x0
* A 6-byte timestamp

## Received Until

> **0xA**

Matches all records that were received by the server before
this value.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xA|              0x0          |
 8  +-------------------------------+
    |  0x0  |     TIMESTAMP         | 
 16 +-------------------------------+
```

* The byte 0xA
* Nine bytes 0x0
* A 6-byte timestamp

## Kinds

> **0xB**

Matches all records which are of one of these kinds.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xB|          0x0          | n |
 8  +-------------------------------+
    |   KIND        |  ...KIND      |
 16 +-------------------------------+
```

* The byte 0xB
* 6 bytes 0x0
* A 1-byte count `n`, then
* A sequence of `n` 4-byte [kinds](kinds.md)

## Tag Values

> **0xC**

Matches all records which have one of these tags.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |0xC|0x0| TTYPE | 0x0       | n |
 8  +-------------------------------+
    |   DATA...                     |
 16 +-------------------------------+
```

* The byte 0xC
* 1 byte 0x0
* A 2-byte [tag type](tag_types.md) in little-endian format
* 3 bytes 0x0
* A 1-byte count `n`, then
* DATA being a sequence of n length-value pairs (length being 1 byte)
  representing the n different values of the tag which cause the filter
  to match. This data is not aligned and tag values are of varying length.
