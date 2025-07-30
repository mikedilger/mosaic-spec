# Mosaic Human Encodings

Human encodings are defined to provide uniform compatible ways for
sharing data between humans and software.

NOTICE: I'm considering switching this to an encoding that includes a checksum.
zbase32 does not (bech32 does). I defined a new one, hum32, but I'm still conflicted
about it.

[zbase32](https://philzimmermann.com/docs/human-oriented-base-32-encoding.txt) is used for human-oriented rendering of binary data.

Prefixes are used to specify the type of data being represented:

| Data                         | prefix |
|------------------------------|--------|
| ed25519 public key, user     | mopub0 |
| ed25519 public key, server   | mosrv0 |
| ed25519 secret key           | mosec0 |
| ed25519 encrypted secret key | mocryptsec0 |
| References                   | moref0 |
| Records                      | morecord0 |

secp256k1 keys are encoded with bech32 as defined by nostr in [NIP-19](https://github.com/nostr-protocol/nips/blob/master/19.md).
