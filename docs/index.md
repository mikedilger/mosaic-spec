# Mosaic

<status>PAGE STATUS: early draft</status>

Mosaic is a *work in progress*.  This specification is incomplete and an early draft.

## Introduction

Mosaic is a
<t>distributed</t> [<sup>rat</sup>](rationale.md#distributed)
and
<t>sovereign</t> [<sup>rat</sup>](rationale.md#sovereign)
social application protocol for the Internet.

Mosaic uses a
<t>client-server</t> [<sup>rat</sup>](rationale.md#client-server)
architecture (as opposed to peer-to-peer).

Mosaic runs over any
<t>duplex communication</t> [<sup>rat</sup>](rationale.md#duplex-communication)
transport protocol that is
<t>TLS</t> [<sup>rat</sup>](rationale.md#tls)
secured such as <t>QUIC</t> [<sup>rat</sup>](rationale.md#quic).

Mosaic does not provide
<t>IP privacy</t> [<sup>rat</sup>](rationale.md#no-ip-privacy)
because this is better provided at a lower network layer.

Mosaic users generate their own identities as
<t>EdDSA ed25519</t> [<sup>rat</sup>](rationale.md#eddsa-ed25519)
keypairs and Mosaic uses a
[master-key subkey](identity.md#master-keys-and-subkeys)
[<sup>rat</sup>](rationale.md#master-key-subkey)
design.

Mosaic [servers](identity.md#users-versus-servers) have key-based
<t>server identites</t> [<sup>rat</sup>](rationale.md#server-identities).

Mosaic identity and endpoint information is [bootstrapped](bootstrap.md) from
<t>Mainline DHT</t> [<sup>rat</sup>](rationale.md#mainline-dht)

Mosaic uses the <t>BLAKE3</t> [<sup>rat</sup>](rationale.md#blake3) hashing function.

Mosaic [records](record.md) are
<t>binary</t> [<sup>rat</sup>](rationale.md#binary-records).

Mosaic [records](record.md) are [editable](reference.md)
if an application wants them to be, as all records have (and can be addressed by)
a unique [hash-based id](reference.md#id-reference)
and a separate [reusable address](reference.md#address-reference).

[Timestamps](timestamps.md) [<sup>rat</sup>](rationale.md#timestamps)
account for <t>leap seconds</t>, have <t>nanosecond accuracy</t>, and extend
out to year 2262.

Clients and Servers
<t>remember the time that records are received</t>
[<sup>rat</sup>](rationale.md#storing-received-at-timestamps).

The Mosaic specification is layered with Core, Transport, Extensions, and
Applications. Only Core and Transport:QUIC are required by all participants.

Mosaic is an offshoot of [nostr](https://github.com/nostr-protocol).

### The name Mosaic

No reason. Just a name. Easy to remember and pronounce. A throwback to
the old NCSA Mosaic browser I suppose. It is not an acronym. We always
capitalize it even in the middle of a sentence.
