# Record

All Mosaic data is stored within Record structures (except for
bootstrap data and protocol messages).

## Maximum Size

The <t>maximum size</t> [<sup>rat</sup>](rationale.md#maximum-size)
of a record is 1 mebibyte (1,048,576 bytes).

## Layout


```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
  0 +-------------------------------+ <-----|
    | Timestamp                     |       |
  8 +-------------------------------+       |
    | Hash 1/5                      |       |
 16 +-------------------------------+       |
    | Hash 2/5                      |  I    |
 24 +-------------------------------+  D    |
    | Hash 3/5                      |       |
 32 +-------------------------------+       |
    | Hash 4/5                      |       |
 40 +-------------------------------+       |
    | Hash 5/5                      |       |
 48 +-------------------------------+ <-----|<--|
    | Unique Address Nonce          |       |   |
 56 +-------------------------------+       |   |
    | Kind                          |   A   |   |
 64 +-------------------------------+   D   |   |
    | Author public key, 1/4        |   D   |   |
 72 +-------------------------------+   R   |   |
    | Author public key, 2/4        |   E   |   |
 80 +-------------------------------+   S   |   |
    | Author public key, 3/4        |   S   |   |
 88 +-------------------------------+       | D |
    | Author public key, 4/4        |       | A |
 96 +-------------------------------+ <-----| T |
    | Signing public key, 1/4       |         A |
104 +-------------------------------+           |
    | Signing public key, 2/4       |         S |
112 +-------------------------------+         E |
    | Signing public key, 3/4       |         C |
120 +-------------------------------+         T |
    | Signing public key, 4/4       |         I |
128 +-------------------------------+         O |
    | Timestamp                     |         N |
136 +-------------------------------+           |
    | Flags                         |           |
144 +-------------------------------+           |
    | LenT  | LenS  | Len P         |           |
152 +-------------------------------+           |
    | Tags...                       |           |
    |                   .. +PADDING |           |
  ? +-------------------------------+           |
    | Payload ...                   |           |
    |                   .. +PADDING |           |
  ? +-------------------------------+<----------|
    | Signature ...                 |
    |                   .. +PADDING |
    +-------------------------------+
```

[Rationale](rationale.md#layout)

## Sections and Fields

Records contain the following sections and fields.

---

### ID

A 48 byte ID from `[0:48]` made up of the following two parts:

#### Timestamp

8 bytes from `[0:8]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

#### Hash

40 bytes at `[8:48]`

This is the first 40 bytes of the BLAKE3 hash of the data section ([48:152+LenT+LenP]).

---

### Data Section

After the ID comes the data section which contains everything else except the
digital signature.

#### Address

A 48 byte Address from `[48:96]` made up of the following three parts.
[<sup>ref</sup>](rationale.md#address-fields)

##### Unique Address Nonce

8 bytes at `[48:56]`. These bytes MUST start with a 1 bit.

These bytes make an address unique (within the context of an author and a
kind). They can be created in a number of different ways, depending on the
application and its purpose:

1. They can be generated randomly.
2. They can be a timestamp, modified with a leading 1 bit.
   This is useful when the addresses should sort in time order.
3. They can be the first 8 bytes of a BLAKE3 hash of a fixed slice
   of bytes. This is useful for applications that require seeking a
   record by a known fixed string of bytes (with known author and kind).
4. They can be copied from a previous record, to replace or version that
   record if the kind is replaceable or versioned.

The method of creation is determined by the application layer.

##### Kind

A [kind](kind.md), represented in 8 bytes at `[56:64]`.

##### Author Public Key

32 bytes at `[64:96]`

This is the identity of the author expressed as their master key.

For ed25519 and secp256k1 it is the master key public key itself.

For post-quantum cryptosystems with larger public keys, it is a 32 byte BLAKE3
hash of the public key.

#### Signing Public Key

32 bytes at `[96:128]`

This is the public key of the signing keypair.

The signing keypair is usually a subkey under the author's master keypair, but
theoretically could be delegated in some other fashion in the future.  For
some records, it is the master public key itself.

For post-quantum cryptosystems with larger public keys, it is a 32 byte BLAKE3
hash of the signing key.

#### Timestamp

8 bytes at `[128:136]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

It is repeated in the ID, but this copy is digitally signed by being included in the data
section and thus hashed, whereas the timestamp in the ID is not hashed or signed and is there
for sorting purposes.

#### Flags

8 bytes at `[136:144]` with the following meanings:

* Byte 0:  Cryptosystem byte
    * bits 0-3 indicate the master key cryptosystem
        * 0 represents the ed25519 cryptosystem
        * 1 represents the secp256k1 cryptosystem with standard mosaic signature
        * 2 represents the secp256k1 cryptosystem using nostr's signature scheme where
          the data is layed out in JSON and signed according to
          nostr [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md).
          See TBD for details of how to embed a nostr event into Mosaic.
        * Other values are currently RESERVED
    * bits 4-7 indicate the signing key cryptosystem, using the same value definitions as for the master key.
* Byte 1:  Various Flags
    * bit 0: `ZSTD` - If 1, the payload is compressed with Zstd
    * bit 1: RESERVED
    * bit 2: `FROM_AUTHOR` - If 1, servers SHOULD only accept the record from the author (requiring authentication)
    * bit 3-5: RESERVED
* Bytes 2-7: RESERVED

All reserved bits SHOULD be 0.
If any reserved bits in bytes 0-3 are set, software MUST reject the record.
If any reserved bits in bytes 4-7 are set, software SHOULD ignore them and accept the record.

#### LenT

2 bytes at `[144:146]` representing the exact length of the tags section in bytes
(not counting padding) as an unsigned integer in little-endian format.

The maximum tags section length is 65536 bytes.

##### LenTPad

This value is NOT included in the record, it is calculated.

The length of the tags section including padding is called `LenTPad` and is
calculated as `(LenT + 7) & !7`

#### LenS

2 bytes at `[146:148]` representing the exact length of the signature section in bytes
(not counting padding) as an unsigned integer in little-endian format.

The maximum signature section length is 65536 bytes.

For ed25519 and secp256k1 cryptosystems, the signature section will be 64 bytes long.
For post-quantum schemes, the signature length will depend on the precise scheme.

##### LenSPad

This value is NOT included in the record, it is calculated.

The length of the signature section including padding is called `LenSPad` and is
calculated as `(LenS + 7) & !7`

#### LenP

4 bytes at `[148:152]` representing the exact length of the payload section in bytes
(not counting padding) as an unsigned integer in little-endian format.

The maximum payload section length is whatever fits such that the entire record
remains within 1,048,576 bytes.

##### LenPPad

This is NOT included in the record, it is calculated.

The length of the payload section including padding is called `LenPPad` and is
calculated as `(LenP + 7) & !7`

#### Tags

Varying bytes at `[152:152+LenT]`

These are searchable key-value [tags](tags.md).

The tags section is padded out to 64-bit alignment.

The maximum tags section length is 65536 bytes.

Tag types are documented at the [Tag Type Registry](tag_types.md)
which includes the defined [Core Tags](core_tags.md).

#### Payload

Varying bytes at `[152+LenTPad:152+LenTPad+LenP].`

Payload is application specific data, and is opaque at this layer of specification,
except for records defined by the Core Application.

The payload section is padded out to 64-bit alignment.

The maximum payload section length is 1_048_384 bytes (which is the maximum record
size minus the header size).

### Signature

Varying bytes at `[152+LenTPad+LenPPad:152+LenTPad+LenPPad+LenS].`

This is the signature of the full hash of the data section, produced according to the
cryptosystem and the [construction](#construction) procedure.

# Construction

## Fill in the Data Section

Fill in the data section from `[48:152+LenTPad+LenPPad]`.

LenS should be a constant determined from the cryptosystem of the signing key.

## Hash the Data Section

Take a BLAKE3 hash of the data section from `[48:152+LenTPad+LenPPad]`,
unkeyed, and extend this to 64 bytes of output using BLAKE3's
`finalize_xof()` function. This 64 byte (512-bit) hash does not directly
go into the record.

## Create and Attach a signature

Generate pre-hashed signature of the 64 byte (512-bit) BLAKE3 hash
generated in step 2 using the Signing secret key, according to its
cryptosystem. Place the signature into the signature section.

## Fill in the ID

* Copy the first 40 bytes of the hash generated earlier to `[8:48]`.
* Copy the 64-bit timestamp from `[128:136]` to `[0:8]`.

# Validation

Records MUST be fully validated by clients.

Records MUST be fully validated by servers upon receipt except for the
steps marked CLIENTS ONLY.

Validation steps:

1. The length must be between 152 and 1048576 bytes.
2. The length must equal 152 + LenTPad + LenPPad + LenSPad.
3. LenS must be correct for the cryptosystem specified by the flags
4. The Signing public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
5. The Author public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
6. CLIENTS ONLY: The Signing public key MUST be verified to be
   a non-revoked subkey of the Author via the Author's
   [bootstrap](bootstrap.md).
7. Take a BLAKE3 hash of the data section from `[48:152+LenTPad+LenPPad]`,
   unkeyed, and extend this to 64 bytes of output using BLAKE3's
   `finalize_xof()` function.
8. Verify the hash: Compare bytes `[0:40]` of this hash with bytes
   `[8:48]` of the record. They must match or validation has failed.
9. Verify the ID timestamp: bytes `[0:8]` must be equal to bytes `[128:136]`.
10. Verify the signature: The signature must be valid for the cryptosystem
    specified in the flags for the signing key.
11. Reserved flags in flag bytes 4-7 must be 0, or else the record should
    be rejected.
12. CLIENTS ONLY: Application specific validation SHOULD be performed.
