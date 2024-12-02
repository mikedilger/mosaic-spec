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
record layout at `[160:208]` making up 48 bytes.

* The the author's public key (32 bytes),
* The timestamp (6 bytes),
* The nonce (6 bytes
* The kind (4 bytes).

An author can replace a record by creating a new record with the same address,
in which case the address is copied (the nonce is not randomly generated).
Replaced records must then contain the same author key, the same timestamp,
and the same kind.  They may however be signed by a different
signing keypair or have their flags modified, their tags changed, and their
content changed.

Rationale:

* By containing the Author public key, record location can be determined through
  [bootstrapping](bootstrap.md).
* These are 48 bytes long and easily fit into a 253 byte tag when needed.
* By not including the hash of content, records can be edited and replaced by
  the author (where edits make sense)
* By containing the kind, records that are edited cannot change their kind.
* By containing the kind, software can filter records that are not
  relevant to a situation without needing to look them up first.
* 48 bits of randomness (in the nonce) is unique enough. The odds that a user
  will have multiple clients that simultaneously create records at the same
  millisecond and also choose the same 48-bit random number is exceedingly low
  given that the odds of choosing the same 48-bit random number is about 1 in
  281 trillion. Normally people use 64-bit random numbers for global
  uniqueness, but we don't need global uniqueness, just unique for the given
  author at that precise millisecond in time.
* These addresses sort in chronological order, per author.
