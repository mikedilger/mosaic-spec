# Identity

<status>PAGE STATUS: early draft</status>

## Users and Servers

Identities are split between Users and Servers.

Users in Mosaic are self-created and self-administered. This puts each user in
control of their own account, allowing them to digitally sign their content,
move to different servers and not to rely on any central service or authority.

This however also requires each end user to securely manage their private key
material. Convenient methods for doing so, as well as recovery, are outside of
the scope of Mosaic except insomuch as we define master keys and subkeys with
the purpose that subkeys are intended for online use, and master keys are
intended to be long-term and kept more securely, perhaps being offline, in
hardware, or managed by a trusted service.

## Public key cryptosystem keypair

Identities are realized as a keypair produced within a public key cryptosystem.
We use the EdDSA ed25519 cryptosystem for digital signature.
See [cryptography](cryptography.md).

A user is identified by their ed25519 master public key.

## Master keys and Subkeys

Users may have subsidiary public keys, known as `subkeys`. At times this may
also be called `signing keys` or `device keys`.

The purpose of subkeys is for online usage in less secure environments, where
compromise and revocation do not invalidate the master key identity that the
user is known by.

Subkeys also support alternative algorithms, such as X25519 public keys for
receiving encrypted information, or nostr secp256k1 keys for backwards
compatibility with nostr.

Users publish their subkeys in a [key schedule record](keyschedule.md), defined
within the core records specification.

A limited number of low-frequency operations in Mosaic require a signature from
the master key. These include (presently):

* Publishing/modifying a [User Bootstrap](bootstrap.md) listing their servers
* Publishing/modifying a user's [Key Schedule](keyschedule.md) with new keys and/or revocations.
* Publishing/modifying a user's [Profile](profile.md) record.

Subkeys might be deterministically derived from the master private key, or they
might not. Nothing in the Mosaic spec requires such, but some implementations
may make use of this.
