# Mosaic Core Tags

<status>PAGE STATUS: early draft</status>

Tag types are 2-byte (16-bit) unsigned integers in little-endian format.

## Notify Public Key

> **0x1**

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x1  |        0x0            |
 8  +-------------------------------+
    | PUBLIC KEY 1/4                |
 16 +-------------------------------+
    | PUBLIC KEY 2/4                |
 24 +-------------------------------+
    | PUBLIC KEY 3/4                |
 32 +-------------------------------+
    | PUBLIC KEY 4/4                |
 40 +-------------------------------+
```

`PUBLIC KEY` is a 32-byte public key.

Records with this tag indicate that the record os if interest to the
person identified by that public key (as their master key).  Being tagged
as such, it should be delivered to all of this persons' INBOX servers as
specified in their [bootstrap](bootstrap.md) record.

## Reply by Hash

> **0x2**

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x2  |  0x0  |     KIND      |
 8  +-------------------------------+
    | HASH 1/4                      |
 16 +-------------------------------+
    | HASH 2/4                      |
 24 +-------------------------------+
    | HASH 3/4                      |
 32 +-------------------------------+
    | HASH 4/4                      |
 40 +-------------------------------+
```

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
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x3  |  0x0  |     KIND      |
 8  +-------------------------------+
    | ADDR 1/6                      |
 16 +-------------------------------+
    | ADDR 2/6                      |
 24 +-------------------------------+
    | ADDR 3/6                      |
 32 +-------------------------------+
    | ADDR 4/6                      |
 40 +-------------------------------+
    | ADDR 5/6                      |
 40 +-------------------------------+
    | ADDR 6/6                      |
 40 +-------------------------------+
```

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
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x4  |  0x0  |     KIND      |
 8  +-------------------------------+
    | HASH 1/4                      |
 16 +-------------------------------+
    | HASH 2/4                      |
 24 +-------------------------------+
    | HASH 3/4                      |
 32 +-------------------------------+
    | HASH 4/4                      |
 40 +-------------------------------+
```

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
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x5  |  0x0  |     KIND      |
 8  +-------------------------------+
    | ADDR 1/6                      |
 16 +-------------------------------+
    | ADDR 2/6                      |
 24 +-------------------------------+
    | ADDR 3/6                      |
 32 +-------------------------------+
    | ADDR 4/6                      |
 40 +-------------------------------+
    | ADDR 5/6                      |
 40 +-------------------------------+
    | ADDR 6/6                      |
 40 +-------------------------------+
```

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

## Quote by Hash

> **0x6**

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x6  |  0x0  |     KIND      |
 8  +-------------------------------+
    | HASH 1/4                      |
 16 +-------------------------------+
    | HASH 2/4                      |
 24 +-------------------------------+
    | HASH 3/4                      |
 32 +-------------------------------+
    | HASH 4/4                      |
 40 +-------------------------------+
```

This is a quote of another record.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one quotes. Quotes are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`HASH` is a 32-byte hash [reference](reference.md) to some other record
indicating which other record this record quotes.

Records with this tag can also have reply and root tags, but not to the
same record that is quoted.

## Quote by Addr

> **0x7**

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |  0x7  |  0x0  |     KIND      |
 8  +-------------------------------+
    | ADDR 1/6                      |
 16 +-------------------------------+
    | ADDR 2/6                      |
 24 +-------------------------------+
    | ADDR 3/6                      |
 32 +-------------------------------+
    | ADDR 4/6                      |
 40 +-------------------------------+
    | ADDR 5/6                      |
 40 +-------------------------------+
    | ADDR 6/6                      |
 40 +-------------------------------+
```

This is a quote of another record.

`KIND` is a 4-byte record [kind](kinds.md) indicating the kind of record
that this one quotes. Quotes are application-independent and may
reference records of any type. This information is provided to prevent
lookup of records of kinds that software is not able to or does not wish
to handle.

`ADDR` is a 48-byte address [reference](reference.md) to some other
record indicating which other record this record quotes.

Records with this tag can also have reply and root tags, but not to the
same record that is quoted.
