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
  0 +-------------------------------+ <-----+
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
 48 +-------------------------------+ <-----|<--+
    | Unique Address Nonce          |       |   |
 56 +-------------------------------+       |   |
    | Kind                          |   A   |   |
 64 +-------------------------------+   D   |   |
    | Author public key, 1/4        |   D   |   |
 72 +-------------------------------+   R   |   |
    | Author public key, 2/4        |   E   |   |
 80 +-------------------------------+   S   |   |
    | Author public key, 3/4        |   S   |   |
 88 +-------------------------------+       |   |
    | Author public key, 4/4        |       |   |
 96 +-------------------------------+ <-----+   |
    | Signing public key, 1/4       |           |
104 +-------------------------------+      S    |
    | Signing public key, 2/4       |      I    |
112 +-------------------------------+      G    |
    | Signing public key, 3/4       |      N    |
120 +-------------------------------+      E    |
    | Signing public key, 4/4       |      D    |
128 +-------------------------------+           |
    | Timestamp                     |      D    |
136 +-------------------------------+      A    |
    | Flags                         |      T    |
144 +-------------------------------+      A    |
    | LenT  | LenS  | LenP          |           |
152 +-------------------------------+           |
    | Tags...                       |           |
    |                   .. +PADDING |           |
152 + LenTPad ----------------------+           |
    | Payload ...                   |           |
    |                   .. +PADDING |           |
152 + LenTPad + LenPPad ------------+ <---------+
    | Signature...                  |
    |                   .. +PADDING |
152 + LenTPad + LenPPad + LenSPad --+
```

[Rationale](rationale.md#layout)

## Sections and Fields

Records contain the following sections and fields:

### ID Section

A 48 byte ID from `[0:48]` made up of the following two parts:

#### ID: Timestamp

8 bytes from `[0:8]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

This is copied from the internal Timestamp from [128:136].

#### ID: Hash

40 bytes from `[8:48]`

This is the first 40 bytes of the BLAKE3 hash of the "Signed Data" section.

### Address

A 48 byte Address from `[48:96]` made up of the following three parts.
[<sup>ref</sup>](rationale.md#address-fields)

#### Address: Unique Nonce

8 bytes at `[48:56]`.  These bytes MUST start with a 1 bit.

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

### Address: Kind

A [kind](kind.md), represented in 8 bytes at `[56:64]`.

### Address: Author Public Key

32 bytes at `[64:96]`

This is the identity of the author, expressed as a public key from their master
EdDSA ed25519 keypair.

### Signing Public Key

32 bytes at `[96:128]`

This is the public key of the signing keypair, which is usually a subkey under
the author's master keypair (but theoretically could be delegated in some other
fashion in the future). It could also be the Master key itself.

### Timestamp

8 bytes at `[128:136]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

It is repeated in the ID, but this copy is digitally signed by being included in the data
that gets hashed.

### Flags

8 bytes at `[136:144]`.

#### Flag Byte 0

* `0x01 ZSTD` - The payload is compressed with Zstd
* `0x04 FROM_AUTHOR` - Servers SHOULD only accept the record from the author (requiring authentication)
* `0x80, 0x40` - Signature scheme:
    * `00 - EDDSA` - EdDSA ed25519 (default)
    * `01 - RESERVED FOR SECP256K1` - secp256k1 Schnorr signatures (not yet in use)
    * `10 - RESERVED`
    * `11 - RESERVED`
    * NOTE: This only affects the signing key and the signature. The hash is
      always created with BLAKE3, and the master key is always EdDSA
      ed25519.

* All other bits - RESERVED and MUST be 0. If any of the reserved bits are 1, software MUST
  reject the record.

#### Flag Bytes 1-7

RESERVED

Bytes 1 and 2 MUST be zero and the record MUST be rejected if any of these bits are set. These are
reserved for future feature flags that require understanding by the software.

The remaining Bytes MUST be ignored.

### LenT

2 bytes at `[144:146]` representing the length of the tags section in bytes
as an unsigned integer in little-endian format.

This represents the exact length of the tags section, not counting padding
at the end to achieve 64-bit alignment.

The maximum tags section length is 65536 bytes.

#### LenTPad

This is NOT included in the record, it is calculated.

The length of the tags section including padding is called `LenTPad` and is
calculated as `(LenT + 7) & !7`

### LenS

2 bytes at `[146:148]` representing the length of the signature in bytes
as an unsigned integer in little-endian format.

This represents the exact length of the signature, not counting padding
at the end to achieve 64-bit alignment.

The maximum signature length is 65536 bytes.

For Ed25519 signatures, the length is 64 bytes.

#### LenSPad

This is NOT included in the record, it is calculated.

The length of the signature section including padding is called `LenSPad` and is
calculated as `(LenS + 7) & !7`

### LenP

4 bytes at `[148:152]` representing the length of the payload section in
bytes as an unsigned integer in little-endian format.

This represents the exact length of the payload section, not counting padding
at the end to achieve 64-bit alignment.

The maximum payload section length is 1_048_384 bytes (which is the maximum record
size minus the header size).

#### LenPPad

This is NOT included in the record, it is calculated.

The length of the payload section including padding is called `LenPPad` and is
calculated as `(LenP + 7) & !7`

### Tags

Varying bytes at `[152:152+LenT]`

These are searchable key-value [tags](tags.md).

The tags section is padded out to 64-bit alignment.

The maximum tags section length is 65536 bytes.

Tag types are documented at the [Tag Type Registry](tag_types.md)
which includes the defined [Core Tags](core_tags.md).

### Payload

Varying bytes at `[152+LenTPad:152+LenTPad+LenP].`

Payload is application specific data, and is opaque at this layer of specification,
except for records defined by the Core Application.

The payload section is padded out to 64-bit alignment.

The maximum payload section length is 1_048_384 bytes (which is the maximum record
size minus the header size).

### Signature

Signature bytes at `[152+LenTPad+LenPPad:]`

The signature field is the signature of the record
produced using the [construction](#construction) procedure.


# Construction

1. Fill in all the signed data from `[48:152+LenTPad+LenPPad]`.
2. Take a BLAKE3 hash of `[48:152+LenTPad+LenPPad]`, unkeyed, and extend to 64 bytes
   of output using BLAKE3's `finalize_xof()` function. This does not
   directly go into the record.
3. Generate EdDSA ed25519ph pre-hashed signature of the 512-bit hash
   generated in step 2 using the Signing secret key, and providing the
   context string of "Mosaic". NOTE: ed25519 calls for a SHA-512 hash,
   but we use a BLAKE3 hash instead. Place the signature at bytes `[152+LenTPad+LenPPad:]`.
4. Copy the first 40 bytes of the hash generated in step 2 to `[8:48]`.
5. Copy the 64-bit timestamp to `[0:8]`.

# Validation

Records MUST be fully validated by clients.

Records MUST be fully validated by servers upon receipt except for the
steps marked CLIENTS ONLY.

Validation steps:

1. The length must be between 152 and 1048576 bytes.
2. The length must equal 152 + LenTPad + LenPPad + LenSPad.
3. The Signing public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
4. The Author public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
5. CLIENTS ONLY: The Signing public key MUST be verified to be
   a non-revoked subkey of the Author via the Author's
   [bootstrap](bootstrap.md).
6. Take a BLAKE3 hash of `[48:152+LenTPad+LenPPad]`, unkeyed, and extend to
   64 bytes of output using BLAKE3's `finalize_xof()` function.
7. Verify the hash: Compare bytes `[0:40]` of this hash with bytes
   `[8:48]` of the record. They must match or validation has failed.
8. Verify the ID timestamp: bytes `[0:8]` must be equal to bytes `[128:136]`.
9. Verify the signature: The signature must be a valid signature according
   to the sitnature scheme specified in the record flags.
11. Reserved flags in flag bytes 1 and 2 must be 0.
12. CLIENTS ONLY: Application specific validation SHOULD be performed.
