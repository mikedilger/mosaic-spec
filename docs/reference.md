# References

<status>PAGE STATUS: early draft</status>

A reference is a pointer from one record to another.

Mosaic defines two kinds of references

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

* Unique address nonce (14 bytes) starting with a 1 bit,
* The kind (2 bytes),
* The the author's public key (32 bytes),

An author can replace a record by creating a new record with the same address,
in which case the address is copied (the nonce is not randomly generated).
Replaced records must then contain the same author key and be of the same
kind, and refer to the same unique address nonce.  They may however be signed
by a different signing keypair or have their flags modified, their tags
changed, and their content changed.
