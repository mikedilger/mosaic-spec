# Record

<status>PAGE STATUS: early draft</status>

All Mosaic persistent data is stored within Record structures (except for
bootstrap data).

## Notation

Byte slice notation `[m:n]` indicates the bytes including `m` up to and
including the byte `n-1` but not including the byte `n`. For example `[8:12]`
represents bytes 8, 9, 10 and 11.

Byte slices that are missing a beginning such as `[:64]` start at 0.

Byte slices that are missing an ending such as `[112:]` continue until the
end of the data.

## Maximum Size

The
<t>maximum size</t> [<sup>rat</sup>](rationale.md#maximum-size)
of a record is 1 mebibyte (1,048,576 bytes).

## Layout


```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    | Signature 1/8                 |
 8  +-------------------------------+
    | Signature 2/8                 |
 16 +-------------------------------+
    | Signature 3/8                 |
 24 +-------------------------------+
    | Signature 4/8                 |
 32 +-------------------------------+
    | Signature 5/8                 |
 40 +-------------------------------+
    | Signature 6/8                 |
 48 +-------------------------------+
    | Signature 7/8                 |
 56 +-------------------------------+
    | Signature 8/8                 |
 64 +-------------------------------+ <-----|
    | Timestamp                     |
 72 +-------------------------------+       |
    | Hash 1/5                      |       |
 80 +-------------------------------+       |
    | Hash 2/5                      |  I    |
 88 +-------------------------------+  D    |
    | Hash 3/5                      |       |
 96 +-------------------------------+       |
    | Hash 4/5                      |       |
104 +-------------------------------+       |
    | Hash 5/5                      |       |
112 +-------------------------------+ <-----|
    | Signing public key, 1/4       |
120 +-------------------------------+
    | Signing public key, 2/4       |
128 +-------------------------------+
    | Signing public key, 3/4       |
136 +-------------------------------+
    | Signing public key, 4/4       |
144 +-------------------------------+ <-----|
    | Unique Address Nonce          |       |
152 +-------------------------------+       |
    | Kind                          |   A   |
160 +-------------------------------+   D   |
    | Author public key, 1/4        |   D   |
168 +-------------------------------+   R   |
    | Author public key, 2/4        |   E   |
176 +-------------------------------+   S   |
    | Author public key, 3/4        |   S   |
184 +-------------------------------+       |
    | Author public key, 4/4        |       |
192 +-------------------------------+ <-----|
    | Timestamp                     |
200 +-------------------------------+
    | Flags | LenT  | LenP          |
208 +-------------------------------+
    | Tags...                       |
    |                   .. +PADDING |
  ? +-------------------------------+
    | Payload ...                   |
    |                   .. +PADDING |
  ? +-------------------------------+
```

[Rationale](rationale.md#layout)

## Fields

Records contain the following fields.

### Signature

64 bytes at `[0:64]`

The signature field is the EdDSA ed25519ph signature of the record
produced using the [construction](#construction) procedure.

### ID

A 48 byte ID from `[64:112]` made up of the following three parts.

#### Timestamp

8 bytes from `[64:72]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

This is the first part of the two-part ID.

#### Hash

40 bytes at `[72:112]`

This is the first 40 bytes of the BLAKE3 hash.

This is the second part of the two-part ID.

### Signing Public Key

32 bytes at `[112:144]`

This is the public key of the signing keypair, which is usually a subkey under
the author's master keypair (but theoretically could be delegated in some other
fashion in the future).

### Address

A 48 byte Address from `[144:192]` made up of the following three parts.
[<sup>ref</sup>](rationale.md#address-fields)

#### Unique Address Nonce

8 bytes at `[144:152]`.  These bytes MUST start with a 1 bit.

These bytes make an address unique (within the context of an author and a
kind). They can be created in a number of different ways, depending on the
application and its purpose:

1. They can be generated randomly.
2. They can be a timestamp.
   This is useful when the addresses should sort in time order.
3. They can be the first 8 bytes of a BLAKE3 hash of a fixed slice
   of bytes. This is useful for applications that require seeking a
   record by a known fixed string of bytes (with known author and kind).
4. They can be copied from a previous record, to replace or version that
   record if the kind is replaceable or versioned.

The method of creation is determined by the application layer.

#### Kind

A [kind](kinds.md), represented in 8 bytes at `[152:160]`.

#### Author Public Key

32 bytes at `[160:192]`

This is the identity of the author, expressed as a public key from their master
EdDSA ed25519 keypair.

### Flags

2 bytes at `[192:194]`

* `0x0001 ZSTD` - The payload is compressed with Zstd
* `0x0004 FROM_AUTHOR` - Servers SHOULD only accept the record from the author (requiring authentication)
* `0x0080, 0x0040` - Signature scheme:
    * `00 - EDDSA` - EdDSA ed25519 (default)
    * `01 - RESERVED FOR SECP256K1` - secp256k1 Schnorr signatures (not yet in use)
    * `10 - RESERVED`
    * `11 - RESERVED`
    * NOTE: This only affects the signing key and the signature. The hash is
      always created with BLAKE3, and the master key is always EdDSA
      ed25519.

* All other bits - RESERVED and MUST be 0. If any of the reserved bits are 1, software MUST
  reject the record.

### Timestamp

8 bytes at `[192:200]`

This is the big-endian unsigned 64-bit timestamp as described in [timestamps](timestamps.md),
which represents the number of nanoseconds that have elapsed since 1 January 1970
including within leap seconds.

It is repeated in the ID, but this copy gets digitally signed.

### LenT

2 bytes at `[202:204]` representing the length of the tags section in bytes
as an unsigned integer in little-endian format.

This represents the exact length of the tags section, not counting padding
at the end to achieve 64-bit alignment.

The maximum tags section length is 65536 bytes.

#### LenTPad

This is NOT included in the record, it is calculated.

The length of the tags section including padding is called `LenTPad` and is
calculated as `(LenT + 7) & !7`

### LenP

4 bytes at `[204:208]` representing the length of the payload section in
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

Varying bytes at `[208:208+LenT]`

These are searchable key-value tags.

<t>Tags</t> [<sup>ref</sup>](rationale.md#tags) are a maximum of 256 bytes long each.

All tags are searchable on servers. If an application requires unsearchable tags,
these can be defined within that application's payload.

Tags are laid out as follows:

```text
0           2               3         256 max
+-----------+---------------+----------+
| type      | value_length  | value ...|
+-----------+---------------+----------+
```

Each tag has a 2-byte (16 bit) type `[0:2]`, a 1-byte (8 bit) value length `[2:3]`,
and a value that is at most 253 bytes long `[3:]`.

Tags only have one value.

The tags section is padded out to 64-bit alignment.

The maximum tags section length is 65536 bytes.

Tag types are documented at the [Tag Type Registry](tag_types.md)
which includes the defined [Core Tags](core_tags.md).

### Payload

Varying bytes at `[208+LenTPad:208+LenTPad+LenP].`

Payload is opaque (at this layer of specification) application-specific data.

The payload section is padded out to 64-bit alignment.

The maximum payload section length is 1_048_384 bytes (which is the maximum record
size minus the header size).

# Construction

1. Fill in all the data from `[112:]`.
2. Take a BLAKE3 hash of `[112:]`, unkeyed, and extend to 64 bytes
   of output using BLAKE3's `finalize_xof()` function. This does not
   directly go into the record.
3. Generate EdDSA ed25519ph pre-hashed signature of the 512-bit hash
   generated in step 2 using the Signing secret key, and providing the
   context string of "Mosaic". NOTE: ed25519 calls for a SHA-512 hash,
   but we use a BLAKE3 hash instead. Place the signature at bytes `[0:64]`.
4. Copy the first 40 bytes of the hash generated in step 2 to `[72:112]`.
5. Copy the 64-bit timestamp to `[64:72]`.

# Validation

Records MUST be fully validated by clients.

Records MUST be fully validated by servers upon receipt except for the
steps marked CLIENTS ONLY.

Validation steps:

1. The length must be between 208 and 1048576 bytes.
2. The length must equal 208 + LenTPad + LenPPad.
3. The Signing public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
4. The Author public key MUST be validated according to the
   [cryptography](cryptography.md) key validation checks.
5. CLIENTS ONLY: The Signing public key MUST be verified to be
   a non-revoked subkey of the Author via the Author's
   [bootstrap](bootstrap.md).
6. Take a BLAKE3 hash of `[112:]`, unkeyed, and extend to
   64 bytes of output using BLAKE3's `finalize_xof()` function.
7. Verify the hash: Compare bytes `[0:40]` of this hash with bytes
   `[72:112]` of the record. They must match or validation has failed.
8. Verify the ID timestamp: bytes `[64:72]` must be equal to bytes `[192:200]`.
9. Verify the signature: The signature must be a valid EdDSA ed25519
   signature of the full 64-byte BLAKE3 hash taken in step 6 with the
   signing public key.
10. Bytes 70 and 71 must be 0.
11. Reserved flags must be 0.
12. CLIENTS ONLY: Application specific validation SHOULD be performed.
