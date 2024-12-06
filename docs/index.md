# Mosaic

<status>PAGE STATUS: early draft</status>

## Introduction

Mosaic is a *distributed* *sovereign* *general-purpose* application protocol for the Internet.

**distributed**: There is no central point of failure, or place that can be taken down. There is no central place to "bootstrap" Mosaic (we use Mainline DHT's bootstrapping, there are mutiple and you can run your own).

**sovereign**: All nodes (users/clients and servers) participation is self-managed and nobody can cancel your account. You manage your own keys. Mosaic does not depend on DNS and Mosaic does not depend on Certification Authorities to issue certificates.

**general-purpose**: Although it started as social media, this architecture has been shown to serve many general-purpose applications.

Mosaic is a **work in progress**.  This specification is EARLY DRAFT.

### What Mosaic is not

*Mosaic is NOT peer-to-peer*: <small>Around the turn of the century, a lot of distributed sovereign protocol work focused on peer-to-peer:  Freenet, GnuNET, and later DHTs.</small> As it turns out, peer-to-peer is difficult because most computers are not fully connected to the Internet. And as there is nothing particular difficult in running a server that *is* fully connected to the Internet (given VPS availability), being strictly peer-to-peer doesn't seem advantageous. So we choose the more rock-solid client-server architecture.

*Mosaic does NOT provide IP privacy*: <small>Around the turn of the century, a lot of distributed sovereign protocol work focused on IP privacy: Freenet, GnuNET, etc.</small>  However, Tor took off as a general privacy layer, and other alternatives exist including i2p and VPNs.  Architecturally, it makes sense to separate application layers from privacy layers.  There is no good reason to reinvent another privacy layer since Mosaic can run on top of an already existing privacy layer.

### Where Mosaic came from

Mosaic is an offshoot of [nostr](https://github.com/nostr-protocol).

Like nostr:

* Mosaic uses sovereign user-controlled key pairs
* Mosaic uses WebSockets
* Mosaic uses client-server architecture, since peer-to-peer has connectivity problems
* Mosaic doesn't provide IP privacy

Unlike nostr:

* Mosaic uses different [cryptography](cryptography.md) (EdDSA ed25519 and BLAKE3)
* Mosaic uses [subkeys](identity.md#master-keys-and-subkeys) from the start for better key management
* Mosaic [servers](identity.md#users-and-servers) have keypair-based identities too, so you can be sure that
  you are connecting to the right server. Servers are identified by their
  keypair, not their DNS-based URL.
* Mosaic [WebSockets](websockets.md) uses TLS 1.2 or 1.3 with either
  self-signed certificates or RawPublicKey, so that no dependency is made upon
  self-proclaimed certificate authority companies.
* Client authentication by servers is done at the TLS layer where clients
  also present self-signed or RawPublicKey certificates.
* Mosaic information (server's IP addresses and user's home server information)
  is [bootstrapped](bootstrap.md) from Mainline DHT
* Mosaic [records](record.md) are binary, meaning they are smaller,
  have no parsing overhead, and have no layout or character encoding
  ambiguities. If you think we should have used CBOR or JSON,
  Refer to
  [debate: Binary](https://github.com/SteveFarroll/mosaic-spec/issues/8)
* Mosaic records are [editable](reference.md) if an application wants them to
  be, as all records can be addressed either by their hash-based id (not
  replaceable) or their address (replaceable) and all records have both a
  hash-based id and an address.
* [Timestamps](timestamps.md) account for leap seconds (unlike unixtime) and
  have millisecond accuracy.
* Clients and Servers remember the time that records are received, so that
  key revocation can revoke all records received after a certain time,
  and not rely on the possibly fake timestamps in the records themselves.
* The specification is layered with Core, Transport, Extensions, and
  Applications being separate. Only Core and WebSockets are required by
  all participants.


Terminology differences

* Nostr *events* are akin to Mosaic *records*
* Nostr *relays* are akin to Mosaic *servers*

### The name Mosaic

No reason. Just a name. Easy to remember and pronounce. A throwback to
the old NCSA Mosaic browser I suppose. It is not an acronym. We always
capitalize it even in the middle of a sentence.
