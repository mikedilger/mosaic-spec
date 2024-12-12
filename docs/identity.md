# Identity

<status>PAGE STATUS: early draft</status>

## Users and Servers

Identities are split between Users and Servers.

Users in Mosaic are self-created and self-administered.

This requires each end user to securely manage their private key material.
Convenient methods for doing so are outside of the scope of Mosaic
except insomuch as we define master keys and subkeys with the purpose that
subkeys are intended for online use, and master keys are intended to be
long-term and kept more securely, perhaps being offline, in hardware, or
managed by a trusted service.

## User Identity

A user is defined to be the person, organization, or other entity with knowledge
of the private half of an [EdDSA](cryptography.md#digital-signature-with-eddsa-ed25519)
keypair.  This key pair is considered their *master keypair*.

A user is referenced by the public half of their *master keypair*.

## Master keys and Subkeys

Users may have subsidiary public keys, known as *subkeys*, *signing keys* or
*device keys* (these terms being mostly functionally interchangeable).

The purpose of subkeys is for online usage in less secure environments, where
compromise and revocation do not invalidate the master key identity that the
user is known by.

Subkeys also support alternative algorithms, such as X25519 public keys for
receiving encrypted information, or secp256k1 keys for backwards compatibility
with nostr.

Users publish their subkeys in a [key schedule record](keyschedule.md), defined
within the core records specification.

A limited number of low-frequency operations in Mosaic require a signature from
the master key. These include (presently):

* Publishing/modifying a [User Bootstrap](bootstrap.md) listing their servers (or
  a [Server Bootstrap](bootstrap.md) listing it's endpoints)
* Publishing/modifying a user's [Key Schedule](keyschedule.md) with new keys and/or revocations.
* Publishing/modifying a user's [Profile](profile.md) record.

Subkeys might be deterministically derived from the master private key, or they
might not. Nothing in the Mosaic spec requires such, but some implementations
may make use of this.
