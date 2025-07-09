# References

<status>PAGE STATUS: early draft</status>

A reference is a pointer from one record to another.

Mosaic defines two kinds of references

## The first bit

References can be distinguished by their first bit. Id references start with a 0 bit,
whereas address references start with a 1 bit.

## Id Reference

An <t>id reference</t> is a pointer to an exact record with no provision for
replacement or edits. It contains some *hash* of the message making it
unique. See [record](record.md) for the Id field.

## Address Reference

An <t>address reference</t> [<sup>rat</sup>](rationale.md#references)
is a pointer to a group of records that have the same
address, which usually represent an initial record and it's subsequent
replacements, often (and by default presumably) with the most recent record
superceding the older records, but refer to the specific application.

An address consists of three fields which are contiguous and in order in the
record layout at `[128:176]` making up 48 bytes.

* Unique address nonce (8 bytes) starting with a 1 bit,
* The kind (8 bytes),
* The the author's public key (32 bytes),

The nonce can be created in a multitude of ways depending on the needs of the
application

* Randomly
* As a copy of a previous record's nonce, to replace that record or add a
  new version
* As a well-known string, thus making it easy to find a well-known record

Records with the same address MUST necessarily have the same author and kind,
but they can be signed by a different signing keypair, have modified flags,
and change their tags and content.
