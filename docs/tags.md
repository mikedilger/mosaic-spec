# Tag types

<status>STATUS: early draft</status>

Tag types are 2-byte (16-bit) unsigned integers in little-endian format.

## Public Key (0x1)

Type = 0x1

Value = A public key

This tag indicates that the record is of interest to the person identified by the public key. By
being tagged as such, that person can easily look up such records.

Note that such records should be delivered to all of this persons' INBOX servers as specified in
their [bootstrap](bootstrap.md) record.

## Reply, Hash Reference (0x2)

Type = 0x2

Value = A 32-byte hash [reference](reference.md)

This record replies to (in a threading sense) the referenced record.

Replies are application-independent and may reference records of any type.
If a record includes this tag, it must also include a `Root, Hash Reference` 0x4 tag as well.

## Reply, Address Reference (0x3)

Type = 0x3

Value = A 48-byte address [reference](reference.md)

This record replies to (in a threading sense) the referenced record.

Replies are application-independent and may reference records of any type.
If a record includes this tag, it must also include a `Root, Address Reference` 0x5 tag as well.

## Root, Hash Reference (0x4)

Type = 0x4

Value = A 32-byte hash [reference](reference.md)

This record replies to a thread whose root is the referenced record. This is to support
loading an entire thread in one round trip.

Replies are application-independent and may reference records of any type.
If a record includes this tag, it must also include a `Reply, Hash Reference` 0x2 tag as well.

## Root, Address Reference (0x5)

Type = 0x5

Value = A 48-byte address [reference](reference.md)

This record replies to a thread whose root is the referenced record. This is to support
loading an entire thread in one round trip.

Replies are application-independent and may reference records of any type.
If a record includes this tag, it must also include a `Reply, Address Reference` 0x3 tag as well.

## Quote, Hash Reference (0x6)

Type = 0x6

Value = A 32-byte hash [reference](reference.md)

This indicates another record is quoted, but not as a threaded reply.
Records with this tag can also have reply and root tags, but not to the same record that is quoted.

## Quote, Address Reference (0x7)

Type = 0x7

Value = A 48-byte address [reference](reference.md)

This indicates another record is quoted, but not as a threaded reply.
Records with this tag can also have reply and root tags, but not to the same record that is quoted.
