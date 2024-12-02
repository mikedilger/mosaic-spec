# Mosaic

## Introduction

Mosaic is a *distributed* *sovereign* *general-purpose* application protocol for the Internet.

**distributed**: There is no central point of failure, or place that can be taken down. There is no central place to "bootstrap" Mosaic (other than Mainline DHT's bootstrapping).

**sovereign**: All nodes (users/clients and servers) participation is self-managed and nobody can cancel your account. You manage your own keys. Mosaic does not depend on DNS and Mosaic does not depend on Certification Authorities to issue certificates.

**general-purpose**: Although it started as social media, this architecture has been shown to serve many general-purpose applications.

### What Mosaic is not

*Mosaic is not peer-to-peer*: Around the turn of the century, a lot of distributed sovereign protocol work focused on peer-to-peer:  Freenet, GnuNET, and later DHTs. As it turns out, peer-to-peer is difficult because most computers are not fully connected to the Internet. And as there is nothing particular difficult in running a server that *is* fully connected to the Internet (given VPS availability), being strictly peer-to-peer doesn't seem advantageous. I wouldn't have thought this previously, but nostr has proven this. So we choose the more rock-solid client-server architecture.

*Mosaic does not provide IP privacy*: Around the turn of the century, a lot of distributed sovereign protocol work focused on IP privacy: Freenet, GnuNET, etc.  However, Tor took off as a general privacy layer, and other alternatives exist including i2p and VPNs.  Architecturally, it makes sense to separate application layers from privacy layers.  There is no good reason to reinvent another privacy layer since Mosaic can run on top of an already existing privacy layer.

### Where Mosaic came from

Mosaic is an offshoot of [nostr](https://github.com/nostr-protocol).

Like nostr:

* Mosaic uses sovereign user-controlled key pairs
* Mosaic uses websockets
* Mosaic uses client-server architecture, since peer-to-peer has connectivity problems
* Mosaic doesn't provide IP privacy

Unlike nostr:

* Mosaic uses different cryptography (EdDSA ed25519)
* Mosaic uses subkeys from the start for better key management
* Mosaic servers have keypairs too, so you can be sure you are connecting
  to the right server. Servers are identified by the public key, not their
  DNS-based URL.
* Mosaic information (server's IP addresses and user's servers) is
  bootstrapped from Mainline DHT
* Mosaic records are binary. The minimal Mosaic record is 216 bytes,
  versus the minimal nostr record of 343 byes. The overhead of JSON
  parsing along with it's ambiguity is gone!
* Mosaic records are editable if an application wants them to be, as all
  records can be addressed either by their hash (not replaceable) or their
  address (replaceable) and all records have both a hash and an address.
* Mosaic WebSockets uses TLS 1.2 or 1.3 with either self-signed certificates
  or RawPublicKey, so that no dependency is made upon self-proclaimed
  certificate authority companies.
* Client authentication by servers is done at the TLS layer where clients
  also present self-signed or RawPublicKey certificates.
* Timestamps account for leap seconds (unlike unixtime) and have millisecond
  accuracy.
* Clients and Servers remember the time that records are received, so that
  key revocation can revoke all records received after a certain time,
  and not rely on the possibly fake timestamps in the records themselves.

Terminology differences

* Nostr *events* are called `records`
* Nostr *event IDs* are called `record hashes`
* Nostr *relays* are called `servers`

### Principles of Design

* The protocol must be simple enough for multiple implementations to
  be developed, but simplicity is not the only factor.
* The protocol must be functional enough to support a wide range of
  applications beyond just social media ones.
* It is okay to do things multiple ways so long as there is one default
  that all developers implement, and the rest of the "ways" are optional.
  For example, we can have multiple transports (WebSockets, WebTransport,
  even REST) without all the developers needing to move beyond WebSockets.
