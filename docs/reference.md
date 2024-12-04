# References

<status>PAGE STATUS: early draft</status>

A reference is a pointer from one record to another.

Mosaic defines two kinds of references

## Hash Reference

A `hash` reference is a pointer to an exact record with no provision for
replacement or edits. It is a prefix of the `hash` of the message. Usually
either a 16-byte (128 bit) or 32-byte (256 bit) prefix is used.

## Address Reference

An `address` reference is a pointer to a group of records that have the same
address, which usually represent an initial record and it's subsequent
replacements, often (and by default presumably) with the most recent record
superceding the older records.

An address consists of four fields which are contiguous and in order in the
record layout at `[128:176]` making up 48 bytes.

* The the author's public key (32 bytes),
* The kind (2 bytes),
* The original timestamp (6 bytes),
* The random nonce (8 bytes),

An author can replace a record by creating a new record with the same address,
in which case the address is copied (the nonce is not randomly generated).
Replaced records must then contain the same author key and be of the same
kind.  They may however be signed by a different signing keypair or have
their flags modified, their tags changed, and their content changed.

The timestamp of a replacement record MUST be larger than the original
timestamp in the address.

Rationale:

* By containing the Author public key, record location can be determined
  through [bootstrapping](bootstrap.md).
* At only 48 bytes long, these can easily fit into a 253 byte tag when needed.
* By not including the hash of content, records can be edited and replaced by
  the author (where edits make sense)
* By containing the kind, records that are edited cannot change their kind.
* By containing the kind, software can filter records that are not
  relevant to a situation without needing to look them up first.
* By containing the original timestamp and a 64-bit nonce, it is statistically
  infeasible for two different records to have the same address
  unintentionally. In fact this is overkill, but using anything shorter
  that still aligns at 64-bits ends up being not unique enough.