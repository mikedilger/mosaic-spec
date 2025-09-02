# Key Schedule (kind)

A Key Schedule record lists subkey information and revocation information for a
master key.

A Key Schedule record MUST be considered invalid if it does not conform to this
specification.

A Key Schedule record MUST be considered invalid if the signing key and the
master key are not identical.

## Kind

**Applicaton**: 0 (Mosaic Core)

**App Kind**: 1 (Key Schedule)

**Flags**: 0x0003 = *Replaceable*, *Everybody*, *NotPrintable*

Reprsentation = 0x0000_0000_0001_000e

## Tags

Every subkey listed in a key schedule record MUST have an associated
[subkey tag](core_tags.md#subkey) listing the subkey.

## Payload

The payload is not [human readable](human_readable_content.md).

The payload contains a sequence of records as follows:


```
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    |KIND|0 |   LEN | MARKER| Zeroed|
 8  +-------------------------------+
    |  REVOC TIMESTAMP or ZEROES    |
 16 +-------------------------------+
    | DATA ...                      |
    | DATA ...                  ... |
    +-------------------------------+
```

* `[0:1]` - Kind which must be one of:
    * 1 - Subkey Attestation
    * 2 - Encryption Key Data
* `[1:2]` - Zero
* `[2:4]` - Length of this record in bytes, as a little-endian 16-bit integer.
* `[4:6]` - MARKER is 2-bytes and is one of the following
    * 0x0 - ACTIVE_SIGNING_KEY - A Mosaic ed25519 signing key (subkey) in current use
    * 0x1 - ACTIVE_ENCRYPTION_KEY - A Mosaic X25519 encryption key in current use
        * These keys are used for receiving encrypted data only, not for signing.
        * Generally the secretkey for encryption is distributed to every device
          that needs the ability to view encrypted data. Being a separate subkey
          from the signing keys, it limits the damage from compromise.
   * 0x40 - REVOKED_ALL - All records signed by the key are to be considered
      invalid.
    * 0x41 - REVOKED_PAST - Records signed by the key that were received prior to
      the revocation timestamp (based on when it was received by software and NOT
      based on the date inside of the record) are still considered valid; however,
      records either received after the revocation timestamp, or with a timestamp
      after the revocation timestamp, are considered invalid.
    * 0x4F - OUT_OF_USE - Key is no longer in use (but nothing is revoked). This
      MAY be used for signing keys or encryption keys.
    * 0x80 - ACTIVE_NOSTR_KEY - A nostr secp256k1 subkey
        * This helps support dual-stack software that works with both nostr and
          Mosaic.
* `[6:8]` - Zeroed
* `[8:16]` - REVOC TIMESTAMP is 8-bytes and is in the format described in
    [timestamps](timestamps.md). Timestamp is required for REVOKED ALL and
    REVOKED PAST.  Timestamp is suggested for OUT_OF_USE. Timestamp SHOULD be
    zeroed in all other cases (which is the same as the minimum timestamp).
* `[16:]` - The data, either a Subkey Attestation (if kind is 1) or an Encryption
   Key Data block (if kind is 2).

### Subkey Attestation

A Subkey Attestation is a assertion signed by a signing subkey indicating
that the key considers itself a subkey of a masterkey. It is formatted as
follows:

```
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    | 2c| 28| 98| f5| 8b|ALG| LenS  |
 8  +-------------------------------+
    | SUBKEY    1/4                 |
 16 +-------------------------------+
    | SUBKEY    2/4                 |
 24 +-------------------------------+
    | SUBKEY    3/4                 |
 32 +-------------------------------+
    | SUBKEY    4/4                 |
 40 +-------------------------------+
    | MASTERKEY 1/4                 |
 48 +-------------------------------+
    | MASTERKEY 2/4                 |
 56 +-------------------------------+
    | MASTERKEY 3/4                 |
 64 +-------------------------------+
    | MASTERKEY 4/4                 |
 72 +-------------------------------+
    | Signature                 ... |
    |                           ... |
    +-------------------------------+
```

* `[0:5]` - The hex characters 0x2C, 0x28, 0x98, 0xF5, 0x8B
* `[5:6]` - A byte indicating the signature algorithm:
    * 0 - ed25519 EdDSA
    * 1 - Secp256k1
    * other - RESERVED
* `[6:8]` - Two bytes representing the length of the signature in bytes
    as an unsigned integer in little-endian format.
    Currently this must be 64 for all defined signature schemes.
* `[8:40]` - The subkey
* `[40:72]` - The master key
* `[72:]` - The signature over bytes `[0:72]` using the signature
    algorithm specified in `[5:6]` with the subkey specified at
    `[8:40]`. If the signature length is not a multiple of 8, it
    should be padded with 0s until the next multiple of 8.

### Encryption Key Data

Encryption Key Data is a block of data describing an encryption key. Note that
encryption keys cannot produce signatures and thus cannot attest that they belong
to any particular master key.

It is formatted as follows:

```
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    | d6| da| f0| bd| 33|ALG| zero  |
 8  +-------------------------------+
    | SUBKEY    1/4                 |
 16 +-------------------------------+
    | SUBKEY    2/4                 |
 24 +-------------------------------+
    | SUBKEY    3/4                 |
 32 +-------------------------------+
    | SUBKEY    4/4                 |
 40 +-------------------------------+
```

* `[0:5]` - The hex characters 0xD6, 0xDA, 0xF0, 0xBD, 0x33
* `[5:6]` - A byte indicating the encryption algorithm:
    * 0 - x25519 ECDH
    * 1 - Secp256k1 ECDH
    * other - RESERVED
* `[6:8]` - Zeroed
* `[8:40]` - The subkey

## Server Used

Key Schedule records are to be posted to all of the author's Outbox servers.
