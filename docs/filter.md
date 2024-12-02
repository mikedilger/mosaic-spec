# Filter

<status>PAGE STATUS: early draft</status>

A filter is a binary record that is up to 65536 bytes long maximum.

It is defined as a contiguous sequence of type-value pairs.  Some types
specify a count or length within their value.

## Hash16 - Type 0x1

* The byte 0x1
* A 1-byte count (n), then
* A sequence of n 16-byte, 128-bit hash prefixes.

This matches up to 256 specific records (as hashes are unique).

## Hash32 - Type 0x2

* The byte 0x2
* A 1-byte count (n), then
* A sequence of n 32-byte, 128-bit hash prefixes.

This matches up to 256 specific records (as hashes are unique).

## Author Keys - Type 0x3

* The byte 0x3
* A 1-byte count (n), then
* A sequence of n 32-byte author public keys.

This matches all records authored by up to 256 different users.

## Signing Keys - Type 0x4

* The byte 0x4
* A 1-byte count (n), then
* A sequence of n 32-byte signing public keys.

This matches all records signed by up to 256 different signing keys.

## Timestamps - Type 0x5

* The byte 0x5
* A 1-byte count (n), then
* A sequence of n 6-byte [timestamps](timestamps.md).

This matches all records that have any of these exact timestamps.

## Since - Type 0x6

* The byte 0x6
* A 6-byte timestamp

This matches all records with a timestamp greater than or equal to
its value.

## Until - Type 0x7

* The byte 0x7
* A 6-byte timestamp

This matches all records with a timestamp less than its value.

## Received Ats - 0x8

* The byte 0x8
* A 1-byte count (n), then
* A sequence of n 6-byte [timestamps](timestamps.md).

This matches all records that were received by the server at any of these
exact timestamps.

## Received Since - Type 0x9

* The byte 0x9
* A 6-byte timestamp

This matches all records that were received by the server at or later
than its value.

## Received Until - Type 0xA

* The byte 0xA
* A 6-byte timestamp

This matches all records that were received by the server before
its value.

## Kinds - Type 0xB

* The byte 0xB
* A 1-byte count (n), then
* A sequence of n 4-byte [kinds](kinds.md)

This matches all records which are of one of these kinds.

## Tag Values - Type 0xC

* The byte 0xC
* A 1-byte count (n), then
* A 2-byte [tag type](tag_types.md)
* A sequence of n pairs of (1-byte-length, value) representing the n values
  which cause the filter to match.
