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

The payload contains a sequence of 48-byte subkey records laid out as follows:

```
            1   2   3   4   4   5   6
    0   8   6   4   2   0   8   6   4
 0  +-------------------------------+
    | SUBKEY    1/4                 |
 8  +-------------------------------+
    | SUBKEY    2/4                 |
 16 +-------------------------------+
    | SUBKEY    3/4                 |
 24 +-------------------------------+
    | SUBKEY    4/4                 |
 32 +-------------------------------+
    |MARKER |   RESERVED            |
 40 +-------------------------------+
    |        REVOC TIMESTAMP        |
 48 +-------------------------------+
    | ...                           |
    |                           ... |
    +-------------------------------+
```

`[0:32]` - SUBKEY is the 32-byte subkey

`[32:34]` - MARKER is 2-bytes and is one of the following

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

`[34:40]` - Bytes 34:40 are reserved and MUST be 0.

`[40:48]` - REVOC TIMESTAMP is 8-bytes and is in the format described in
[timestamps](timestamps.md). Timestamp is required for REVOKED ALL and
REVOKED PAST.  Timestamp is suggested for OUT_OF_USE. Timestamp SHOULD be
zeroed in all other cases.

## Server Used

These are posted to all of the author's Outbox servers.
