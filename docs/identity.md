# Identity

## Sovereign Identity

Identities in Mosaic are self-created and self-administered.

This requires each entity (user or server) to securely manage their own
secret key material. Convenient methods for doing so are outside of the
scope of Mosaic.

An identity is defined to be the entity with knowledge of the secret half
of a digital signature cryptosystem signing keypair.

This identifying keypair is herein called their <t>master keypair</t>.

An entity (user or server) is referenced by the public half of their
<t>master keypair</t>.

## Digital Signature Cryptosystems

Mosaic core currently supports two cryptosystems, and a third is expected in the
future:

* <t>ed25519</t>: This is the default and recommended cryptosystem for new users.

* <t>secp256k1</t> This cryptosystem is provided for backwards compatibility with nostr.

## Server Identities

In Mosaic, servers have identities similar to how users have identities.

However, the options for servers are limited. Servers must use the <t>ed25519</t>
cryptosystem, and servers may not have subkeys but rather must use their master
keypair directly.

## Types of Identity

|Entity|Cryptosystem|Subkeys|
|------|------------|-------|
| User | ed25519    | Yes   |
| User | secp256k1  | Yes   |
| Server | ed25519  | No    |

## Master keys and Subkeys

Users MAY have subsidiary public keys, known as <t>subkeys</t>, <t>signing keys</t> or
<t>device keys</t> (these terms being mostly functionally interchangeable).

<t>subkeys</t> can use either cryptosystem for digital signature, or go beyond this
and serve other purposes such as encryption. Cryptosystems for encryption remain TBD.

The purpose of subkeys is primarily to facilitate online usage in less secure
environments without risking exposure of the master key secret, where
compromise and revocation do not invalidate the master key identity that the
user is known by.

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

Currently master keys cannot be revoked, and loss of a master key is catastrophic.
