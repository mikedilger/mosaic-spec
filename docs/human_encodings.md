# Mosaic Human Encodings

Human encodings are defined to provide uniform compatible ways for
sharing data between humans and software.

[bech32](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki#bech32)
is used in many of these due to its superior properties. Refer to the link.

## Keys

User public keys are encoded with bech32 using the `moupub` prefix.

Server public keys are encoded with bech32 using the `mospub` prefix.

Public keys can be encoded with bech32 using the `mopub` prefix if the
encoder does not know whether it represents a user or a server.

Secret keys are encoded with bech32 using the `mosec` prefix.

Hashes are encoded with bech32 using the `mohash` prefix.

Addresses are encoded with bech32 using the `moaddr` prefix.
