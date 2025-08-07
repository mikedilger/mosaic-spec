# Identity

## Sovereign Identity

Identities in Mosaic are self-created and self-administered.
This requires each end user and server to securely manage their secret key material.
Convenient methods for doing so are outside of the scope of Mosaic.

An identity is defined to be the entity with knowledge of the secret half
of an [EdDSA](cryptography.md#digital-signature-with-eddsa-ed25519) keypair.

This identifying keypair is considered their <t>master keypair</t>.

An entity (user or server) is referenced by the public half of their
<t>master keypair</t>.

## Server Identities

In Mosaic, servers have identities similar to how users have identities.

## Master keys and Subkeys

Users MAY have subsidiary public keys, known as <t>subkeys</t>, <t>signing keys</t> or
<t>device keys</t> (these terms being mostly functionally interchangeable).

The purpose of subkeys is primarily to facilitate online usage in less secure
environments without risking exposure of the master key secret, where
compromise and revocation do not invalidate the master key identity that the
user is known by.

Subkeys also support alternative algorithms, such as X25519 public keys for
receiving encrypted information, or secp256k1 keys for backwards compatibility
with nostr.

Users publish their subkeys in a [key schedule record](keyschedule.md), defined
within the core application.

A limited number of low-frequency operations in Mosaic require a signature from
the master key. These include (presently):

* Publishing/modifying a [User Bootstrap](bootstrap.md) listing their servers (or
  a [Server Bootstrap](bootstrap.md) listing it's endpoints)
* Publishing/modifying a user's [Key Schedule](keyschedule.md) with new keys and/or revocations.
* Publishing/modifying a user's [Profile](profile.md) record.

NOTE: Nothing in the Mosaic spec requires that subkeys are deterministically derived
from the master secret key. How subkeys are generated is out of scope.

## Rollover and Revocation

Subkeys can be rolled over (marked "out of use") or revoked. Revocation can apply from
a particular timestamp, or it can cover all records signed with the keypair.

See [Key Schedule](keyschedule.md) for more details.

The use of a [key schedule record](keyschedule.md) by servers is not yet specified
and yet to be worked out. For the moment, servers should use their master key directly.

Currently master keys cannot be revoked, and loss of a master key is catastrophic.
