# Record

<status>PAGE STATUS: draft</status>

A record is a datum within Mosaic. All datums are records.

## Notation

Byte slice notation `[m:n]` indicates the bytes including `m` up to and
including the byte `n-1` but not including the byte `n`. For example `[8:12]`
represents bytes 8, 9, 10 and 11.

## Maximum Size

The maximum size of a record is 1 mebibyte (1,048,576 bytes).

This is specified so that length fields inside of records can be of a defined
fixed number of bits and so that software can make reasonable decisions about
buffer sizes.

## Layout

Note that records are laid out in a way to provide 64-bit alignment.

```text
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    | Signature 1/8                 |
    +-------------------------------+
    | Signature 2/8                 |
    +-------------------------------+
    | Signature 3/8                 |
    +-------------------------------+
    | Signature 4/8                 |
    +-------------------------------+
    | Signature 5/8                 |
    +-------------------------------+
    | Signature 6/8                 |
    +-------------------------------+
    | Signature 7/8                 |
    +-------------------------------+
    | Signature 8/8                 |
 64 +-------------------------------+
    | Hash 1/4                      |
    +-------------------------------+
    | Hash 2/4                      |
    +-------------------------------+
    | Hash 3/4                      |
    +-------------------------------+
    | Hash 4/4                      |
 96 +-------------------------------+
    | Signing public key, 1/4       |
    +-------------------------------+
    | Signing public key, 2/4       |
    +-------------------------------+
    | Signing public key, 3/4       |
    +-------------------------------+
    | Signing public key, 4/4       |
128 +-------------------------------+
    | Author public key, 1/4        |
    +-------------------------------+
    | Author public key, 2/4        |
    +-------------------------------+
    | Author public key, 3/4        |
    +-------------------------------+
    | Author public key, 4/4        |
160 +-------------------------------+
    | Kind  | Orig Timestamp        |
168 +-------------------------------+
    | Nonce                         |
176 +-------------------------------+
    | Flags | Timestamp             |
184 +-------------------------------+
    |AppFlgs| Len_t | Len_p         |
192 +-------------------------------+
    | Tags...                       |
    |                   .. +PADDING |
  ? +-------------------------------+
    | Payload ...                   |
    |                   .. +PADDING |
  ? +-------------------------------+
```

## Fields

Records contain the following fields.

See the [layout](#layout) section for the binary layout of these fields within
a record.

### Signature

64 bytes at `[0:64]`

The signature field is the EdDSA ed25519ph signature using the 64-byte hash at
`[64:128]`. `ph` means "pre hashed" (we will hash the content with BLAKE3 and
tell EdDSA that it is a SHA-512 hash of the message). We also provide a context
to this algorithm of b"Mosaic";

This signature is made with the Signing private key and is represented in 64
bytes.

Rationale:

* EdDSA uses a SHA-512 hashing internal to the algorithm. But BLAKE3 is a faster
  especially for longer messages, and EdDSA works just fine with it.
* We provide a context so that users cannot be tricked by one application into
  signing content for a different application (in case users think they can use
  the same keypair for every application).

### Hash

64 bytes at `[64:96]`

The hashing function is BLAKE3, unkeyed, with 256-bits (32 bytes) of output.

Note that we extend this to 512-bits with `finalize_xof()` for the input of
the EdDSA algorithm, but we only store in the event the 256-bit prefix.

This input of the hash is all the data starting at byte 96, `[96:]`,
everything except the signature and hash itself.

### Signing Public Key

32 bytes at `[96:128]`

This is the public key of the signing keypair, which is usually a subkey under
the author's master keypair (but theoretically could be delegated in some other
fashion in the future). This is represented in 32 bytes (256 bits).

### Author Public Key

32 bytes at `[128:160]`

This is the identity of the author, expressed as a public key from their master
EdDSA ed25519 keypair, which is represented in 32 bytes (256 bits).

### Kind

2 bytes at `[160:162]`

This is the [kind](kinds.md) of the record which is determines the application
this record is part of, which then determines the nature of the non-core tags
and the payload. This is represented in 2 bytes (16 bits) as an
unsigned integer, little-endian.

### Orig Timestamp

6 bytes at `[162:168]`

This is a timestamp represented in 6 bytes (48 bits) according to
[timestamps](timestamps.md).

If the record is not a replacement of another record (the usual case)
this is the same as [Timestamp](#timestamp).

If the record is a replacement of another record, it copies the address
from the original record which will fill this with the original record
timestamp.

### Nonce

8 bytes at `[168:176]` which are randomly generated by the author.
This keeps addresses unique.

### Flags

2 bytes at `[176:178]`

* 0x01 ZSTD - The payload is compressed with Zstd
* 0x02 FROM_AUTHOR - Servers SHOULD only accept the record from the author (requiring authentication)
* 0x04 TO_RECIPIENTS - Servers SHOULD only serve the record to people tagged (requiring authentication)
* 0x08 NO_BRIDGE - Bridges SHOULD NOT propogate the record to other networks (nostr, mastodon, etc)
* 0x10 EPHEMERAL - The record is ephemeral; Servers should serve it to current subscribers and not keep it.
* 0x20 - RESERVED and MUST be 0
* 0x80, 0x40 - Signature scheme:
    * 00 - (default) EdDSA ed25519
    * 01 - (nostr) secp256k1 Schnorr
    * 10 - RESERVED
    * 11 - RESERVED
    * NOTE: This only affects the signing key and the signature. The hash is
      always created with BLAKE3, and the master key is always EdDSA
      ed25519. This enables using nostr keys as subkeys.
* All other bits - RESERVED and MUST be 0

### Timestamp

6 bytes at `[178:184]`

This is a timestamp represented in 6 bytes (48 bits) according to
[timestamps](timestamps.md).

If this record replaces a previous record, this timestamp MUST be larger
than [Orig Timestamp](#orig-timestamp).

### AppFlgs

2 bytes at `[184:186]`

These are bitflags reserved for use by the specific application based on
the [kind](#kind).

### Len_t

2 bytes at `[186:188]` representing the length of the tags section in bytes
as an unsigned integer in little-endian format.

This represents the exact length of the tags section, not counting padding
at the end to achieve 64-bit alignment.

The maximum tags section length is 65536 bytes.

### Len_p

4 bytes at `[188:192]` representing the length of the payload section in
bytes as an unsigned integer in little-endian format.

This represents the exact length of the payload section, not counting padding
at the end to achieve 64-bit alignment.

The maximum payload section length is 1_048_384 bytes.

### Tags

Varying bytes at `[192:192+Len_t]`

These are searchable key-value tags.

Unlike nostr tags, all of thsese are searchable. If an application requires
unsearchable tags, these can be defined within that application's payload.

Tags are laid out as follows:

```text
0           2         3         256 max
+-----------+---------+----------+
| type      | length  | value ...|
+-----------+---------+----------+
```

Each tag has a 2-byte (16 bit) type `[0:2]`, a 1-byte (8 bit) length `[2:3]`,
and a value that is at most 253 bytes long `[3:]`.

Tags only have one value.

The tags section is padded out to 64-bit alignment.

The maximum tags section length is 65536 bytes.

Tag types are documented at [Tag Type Registry](tag_types.md) and
the tags are defined at [Core Tags](core_tags.md).

Rationale:

* Tag values should not be too large as they need to be indexed by relays.
* Constraining the value to 253 bytes allows an entire TLV (with 16-bit
  type and 8-bit length) to fit within 256 bytes.

### Payload

Varying bytes at `[192+Len_t:192+Len_t+Len_p].`

Payload is opaque (at this layer of specification) application-specific data.

The payload section is padded out to 64-bit alignment.

The maximum payload section length is 1_048_384 bytes
