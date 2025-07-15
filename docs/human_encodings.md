# Mosaic Human Encodings

Human encodings are defined to provide uniform compatible ways for
sharing data between humans and software.

[zbase32](https://philzimmermann.com/docs/human-oriented-base-32-encoding.txt) is used for human-oriented rendering of binary data.

Prefixes are used to specify the type of data being represented:

* Public keys for users are prefixed with `mopub0`
* Public keys for servers are prefixed with `mosrv0`
* Secret keys are prefixed with `mosec0`
* Encrypted secret keys are prefixed with `mocryptsec0`
* References are prefixed with `moref0`
* Records are prefixed with `morecord0`
