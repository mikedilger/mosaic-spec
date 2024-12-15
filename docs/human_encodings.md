# Mosaic Human Encodings

Human encodings are defined to provide uniform compatible ways for
sharing data between humans and software.

[bech32](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki#bech32)
is used in many of these due to its superior properties. Refer to the link.

## Keys

Public keys are encoded with bech32 using the `mopub` prefix.

Secret keys are encoded with bech32 using the `mosec` prefix.

References are encoded with bech32 using the `moref` prefix.
