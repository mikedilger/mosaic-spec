# Rationale

For brevity, the rationale for various decisions has been elided from its context and linked
to this page with [<sup>rat</sup>](#) links.

| Contents |
|----------|
| [0-RTT](#0-rtt) |
| [Binary Records](#binary-records) |
| [BLAKE3](#blake3) |
| [Bootstrap Length](#bootstrap-length) |
| [Client-Server](#client-server) |
| [Distributed](#distributed) |
| [Duplex Communication](#duplex-communication) |
| [EdDSA ed25519](#eddsa-ed25519) |
| [Mainline DHT](#mainline-dht) |
| [Master-Key Subkey](#master-key-subkey) |
| [No IP Privacy](#no-ip-privacy) |
| [QUIC](#quic) |
| [Records](#records) |
| [References](#references) |
| [Server Identities](#server-identities) |
| [Sovereign](#sovereign) |
| [Storing received-at timestamps](#storing-received-at-timestamps) |
| [Timestamps](#timestamps) |
| [TLS](#tls) |
| [WebSockets](#websockets) |
| [XOR](#xor) |

---

## 0-RTT

The QUIC transport could have used 0-RTT for higher performance, but 0-RTT has
multiple downsides:

* Requests MUST be idempotent because they are replayable
* Endpoints can be used in attack amplification, as the remote hasn't proven it
  controls the IP address it claims.
* There is no forward secrecy.

---

## Binary Records

The binary record format has been designed for performance and size.

* The on-network format can be identical to a client's or server's on-disk format
* There is no parsing overhead (JSON parsing, for example, is quite expensive)
* Signing and verification are in-place and require zero data copies.
* Fields from the binary record can be accessed directly without deserialization,
  meaning there is optionally no deserialization or copying needed.
* Data is aligned to 8-byte boundaries
* Endian order is well specified

Here is a comparison to JSON and CBOR (or BEVE or similar).

|Encoding|Human Readable|Extensible|Parse Speed|Copy to Sign|Size     |
|--------|--------------|----------|-----------|------------|---------|
|JSON    | **yes**      | **yes**  | slow      | copy       | largest |
|CBOR    | no           | **yes**  | **fast**  | Maybe      | **medium** |
|Binary  | no           | no       | **NONE**  | **NO**     | **SMALLEST** |

Except for human readability, CBOR is better than JSON in cases where extensibility
is needed.

Other than that, binary is better in all respects.

We argue that extensibility of the main record is sufficiently done via record kinds,
tags and content, and therefore extensibility is not needed for this structure (nostr
records have not changed since they were introduced, even though there have been
reasons to do so, and they are even simpler).

We sacrifice:

* Human readability
* Extensibility

We gain:

* Zero parsing required
* Zero copying for signing required
* Smallest size has been achieved

BTW: JSON also has character encoding ambiguities.

### Human Readability

There is a push to use a human-readable format. However, while you can take a
high performance binary format and make it human-readable, you cannot take a
human-readable format and make it high performance. So to satisfy the requirements
of the most number of people, we have to use a high performance binary format
within the core of the protocol.

This shouldn't be a big concern. It hasn't been a big concern in other widely adopted
protocols. For example, HTTP that is transported with gzip is binary data. Developers
never see the gzipped data, all the tools they work with show them the line-oriented
HTTP after gunzip. I am not aware of anybody complaining that gzipped HTTP is binary,
and a bad choice because it is not human readable. I believe our situation is a close
parallel to this one.

Even though this specification specifies the binary encoding of everything, developers
can just use a library that serializes and deserializes into a code structure, and just
deal with that structure.

---

## BLAKE3

This is a very fast hash function with 128-bit security. It is even sometimes faster in
software than hardware versions of SHA-256, and s about 14x as fast as software versions
of SHA-256. It is highly parallelizable and can take advantage of vector instructions.

It's predecessor BLAKE was the most analyzed algorithm during the NIST SHA-3 competition.

EdDSA ed25519 is defined to use SHA-512 and we are using BLAKE3 as a drop-in replacement, so
we have to produce 512-bit hashes. But these   do not need to be in the record. What the
record needs is some way to index and reference it, and part of the hash (at least 256 bits)
is good enough for that purpose.

In BLAKE3, hashes are variable size, and smaller outputs are prefixes of longer outputs.
We utilize this.

---

## Bootstrap Length

Bootstraps are limited to 983 characters because ed25519 bootstraps are stored in the
bittorrent mainline (Kademlia) DHT which limits records to 1000 bytes, which includes
the salt, prefix and bencoding overhead.

---

## Client-Server

Mosaic uses a client-server architecture.

This is because clients and servers serve different roles.
Clients are an interface to the user, while servers are an interface to shared Internet
accessible data.

Nothing precludes a client from also being a server, and calling itself a node.

Nothing precludes servers running behind NAT and being hole-punched into, although such
a mechanism is not provided for currently (Iroh does a bit more than we need and isn't
AFAIK an open protocol but a single software stack).

Mosaic servers are used in a way that presumes they will be online and available most of
the time. Users functionally declare where their data is, and that "where" needs to be
available to other users over the Internet most of the time.

Mosaic has built-in redundancy across such servers, but nonetheless a machine
that is not even intended or capable of being up all the time (such as a phone or
laptop) makes a poor mosaic server.

---

## Distributed

Centralized systems have a fatal flaw: they can easily be captured and controlled by people you do not
agree with such as corporate or government censors or hackers. By having a
central point of attack, the entire
system can be taken down or damaged by bad actors.

Mosaic is desgined to be fully distributed, with no centralized points of failure.

There is no central place to bootstrap Mosaic either. We use Mainline DHT's bootstrapping, there are
mutiple bootstrap servers and you can run your own.

---

## Duplex Communication

We base Mosaic on transport protocols that provide duplex communication such as QUIC and Web
Sockets. This is in opposition to a request-response architecture such as HTTP REST.

This allows either side to initiate an action, which turns out to be a very useful feature.
For example when a new record that matches a user's subscribed filters arrives at the server,
the server can immediately shuttle it to the client. In a request-response architecture, there
would be a delay until the client polled again. And due to the polling nature, the client would
need to make many poll requests even when nothing was ready for them.

---

## EdDSA ed25519

We chose this elliptic curve because it is the state of the art.

* It is fast
* It is space efficient
* It is widely studied
* It has very good resistance to side-channel attacks
* It does not require point validation
* It interoperates with ssh, gpg, TLS, Mainline DHT, and other modern ed25519-based
  identity systems

We replaced the hashing algorithm because multiple good cryptographers have indicated that any
secure hashing algorithm that produces hashes of the right size could be used, and we chose
[BLAKE3](#blake3).

We provide a context string so that users cannot be tricked by one application into signing
content for a different application (in case users think they can use the same keypair for
every application).

We added additional checks in order to provide the following guarantees:

* Existentially and Strongly unforgeable under chosen message attacks
* Strongly Binding Signature

See [Taming the many EdDSAs](https://eprint.iacr.org/2020/1244.pdf)

See [The Provable Security of Ed25519: Theory and Practice](https://eprint.iacr.org/2020/823.pdf)

---

## Mainline DHT

Mainline DHT is distributed and censorship resistant, including being resistant to (or able
to detect) Sybil attacks. It is the largest (or one of) and longest living DHT.

It also has this mutable data functionality and works with ed25519 signed data.

### Salt

We use the salt to avoid collisions, in case the same ed25519 identity keypair is used by
both mosaic and [pubky](https://github.com/pubky), or in case we need to change the format
of the bootstrap in the future.

We also need different salts because we have two different kinds of bootstraps already.
Different salts allow a server keypair to also be a user keypair without collision.

These salts are short enough to not use too much space.

The user bootstrap salt used to be `mub24` but was changed to `mub25` when the format of
printable public keys changed.

### Sequence Numbers

We don't need collision protection here because writing this record requires the secure
master key which is likely to be in only one place, and even if copied around we still
do not expect people to write often from multiple places.

We could change to timestamps instead if this becomes a problem and still remain backwards
compatible.

---

## Master-Key Subkey

The primary purpose was to allow users to have per-device online keys whil keeping their
master secret offline.

However this design facilities quite a bit more functionality such as:

* Separate encryption keys
* Subkeys using alternate cryptosystems (e.g. secp256k1 nostr keys)
* Subkeys as personas, to post about very different things and let people easily follow only
  some of what you post.

---

## No IP Privacy

Several IP privacy network layers exist which already do a good job (Tor, i2p, VPNs). The IP privacy
issue is almost entirely orthogonal to the application protocol, and so architecturally it makes sense
to separate application layers from privacy layers. There is no good reason to reinvent another privacy
layer just for Mosaic since Mosaic can run on top of an already existing general-purpose privacy layer.

---

## QUIC

HTTP has a lot of baggage. WebSockets layers on top of that.

What we fundamentally need is size-unrestricted datagrams within TLS.  QUIC
gives us that if we do our own framing within a QUIC stream.

QUIC is widely deployed due to its use in HTTP 1.3. QUIC is also much
faster due to fewer roundtrips and no head-of-line blocking, while
taking care of TLS, multiplexed streams, congestion control and other
details that we would not dare to reimplement.

QUIC also allows us to use ed25519 keys within TLS and get better assurance
for a client that it is talking to the right server.

However QUIC with modified TLS characteristics is unlikely to work in a browser
based client. And QUIC does not work over Tor which is TCP based. Thus we
must support other transports.

---

## Records

### Maximum Size

This is specified so that length fields inside of records can be of a defined fixed number
of bits and so that software can make reasonable decisions about buffer sizes.

### Layout

Record layout provides a compromise between being tightly packed and providing 64-bit
alignment of fields.

All data that needs hashing is contigous so that no data needs to be copied in order to produce
the hash. The hash-based id and signature could have appeared before or after, and we have chosen
to place them before to give them well defined offsets.

### Flags

A number of record properties that span applications have been conceived of and we
provided space to set these.

### Duplicate Timestamp

We have the timestamp twice.

We have it in the ID so that IDs sort in timestamp order.

But IDs are not hashed and signed. So we have to duplicate it in the main part of the record.

### Tags

Tag values usually don't need to be very large. However occasionally they do, such as when
referencing a URL. Thus we wish to have the possibility of very large tags. A two-byte
length gives us 65,536 bytes which seems like plenty.

Although tags must be indexed by storage engines, storage engines MAY simply index a fixed
prefix of a tag rather than the entire thing as indexes do not need to be perfect (records
which do not actually match can be efficiently removed after the index lookup operation).

While for some tag types a length could be inferred, this is not true in general. Applications
should not be required to recognize every tag type to look up its known length or type-specific
method of length calculation.

Some tag types MAY start with padding in the value in order to better align their data.

---

## TCP

TCP with TLS 1.3 is a required transport so that servers may operate through UDP-adverse
situations. It is technically simpler than QUIC and not much of an additional ask.

---

## References

### Id Fields

The hash in the ID cryptographically represents all the data that was hashed.
It is 40 bytes long because we needed at least 32 bytes for cryptographic
reasons, and because addresses are already 48 bytes long, we used the extra
8 bytes to extend the hash.

The entire 512-bit (64 byte) hash is overkill. Even though we have to produce
it for the EdDSA ed25519 signature, we don't have to waste precious record
space with that entire long hash.

By putting the big-endian timestamp at the front of the ID, IDs sort in time
order and group temporally (for database performance).

### Address Fields

By not including the hash of content, records can be edited and replaced by
the author (where edits make sense). Application will specify which kind of
reference to use in which context.

By containing the Author public key, record location can be determined
through [bootstrapping](bootstrap.md).

By containing the kind, records that are edited cannot change their kind.

By containing the kind, software can filter records that are not relevant to
a situation without needing to look them up first.

By containing a long unique nonce, it is statistically infeasible for two
different records to have the same address unintentionally. In fact this is
overkill, but we had the space due to alignment requirements.

By not specifying how the nonce is generated, applications can solve different
problems with different requirements.

At only 48 bytes long, these can easily fit into a 253 byte tag when needed
and are the same size as an ID.

By the first bit being a 1, IDs and ADDRs can be intermixed and their type
determined by that bit.

--

## Server Identities

By recognizing a server by it's key, we can verify server identity directly.

* We no longer need to trust DNS, that it gave us the right IP address
* We no longer need to trust Certification Authorities, that the DNS to IP address binding
  is correct.

Additionally, servers can now present themselves at multiple transport endpoints. They can
also hop from endpoint to endpoint if they are being actively targetted.

By having a strong form of identification, clients can maintain reliable reputational
information about servers even as they move to different endpoints.

---

## Sovereign

Ethically, we believe in free speech and empowering individuals.

All participation is self-managed and nobody can cancel your account. You manage your own keys.

This allows users to be independent of any service provider and to change the servers they use at
any time.

Mosaic does not depend on upstream DNS providers and Mosaic does not depend on Certification Authorities
to issue certificates.

We use a dumb-server smart-client model in order to push as much control into the user's realm (the client
space) as feasible.

---

## Storing received-at timestamps

We require clients and servers to store this information for two reasons.

The first and most important is for key revocation. Revocation can't be applied to only
records in the past judged by timestamps in the records themselves because those
timestamps can be forged. By remembering when a record was received, an upper time
bound is established, and records prior to that bound can continue to be trusted while
records after that bound can be invalidated.

Secondly, when synchronizing with a relay, clients want to ask for records they
have not gotten yet. They can use this received-at data to avoid missing records
that were timestamped earlier but arrived later than the last time they checked for
records.

---

## Timestamps

### Leap Seconds

Unixtime is a discontinous time scale that is occasionally adjusted by leap seconds.
Subtracting two unixtimes could give a time interval that is off by up to 28 seconds
(for example when comparing dates before 1 Jan 1972 with today).

### Nanoseconds

32 bytes do not provide enough space for sub-second precision. Once you choose to use
the next larger integer (u64) you get plenty of numeric space. Thus we use it for
nanosecond precision, even if nobody needs to be that precise.

### Big Endian

Timestamps are stored in big-endian format so that they sort in time order even when
they are interpreted as a simple sequence of bytes (lexigraphically).

### Timestamp Ranges

Timestamp ranges are inclusive on both ends. This is because they are usually iterated
backwards (from the newest to the oldest) and if we made the oldest one exclusive (as
a normal range would do) it would easily suprise many people who would expect the
until to be the exclusive one. Also, timestamps are fine enough that including one extra
nanosecond has little practical effect.

---

## TLS

We use TLS security because it is heavily researched and state-of-the-art.

This next section applies to QUIC and TCP, but not to WebSockets where browsers
will enforce security policies.

In the QUIC and TCP transports we don't use the X.509 certificates as intended since both
client and server are known by their public keys, not by DNS names.  Instead we use
RawPublicKey or self-signed certificates. Clients are authenticated via client-side
certificates.

---

## WebSockets

Browser-based JavaScript (and WASM) clients are bound by tight security policies that do
not allow TLS manpulation of QUIC connections. Therefore we cannot use TLS authentication
via self-signed certificates, nor can we expect ed25519 algorithms to be supported by the
browsers (they are not supported as of this writing by any major browser).

Additionally, QUIC based on UDP does not work over Tor which is based on TCP.

So for these clients, an alternate form of server-to-client authentication must be made
available, and the WebSocket transport fits this bill.

---

## XOR

We use XOR within the Encrypted Secret Key algorithm. XOR is very efficient. But XOR
is dangerous if used incorrectly. We think it is ok for the following reasons, but will
change the algorithm if it can be shown to be weak:

* The symmetric key is random.
* The symmetric key is non-deterministic (not even by the password since we use a salt).
* The symmetric key is the full length of the data being encrypted (not repeated).
* The data being encrypted is short (32 bytes) and random, and not subject to frequency
  analysis.
* There is no known-plaintext attack possible, so the malleability is not relevant.
