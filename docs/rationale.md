# Rationale

For brevity, the rationale for various decisions has been elided from its context and linked
to this page with [<sup>rat</sup>](#) links.

| Contents |
|----------|
| [Binary Records](#binary-records) |
| [BLAKE3](#blake3) |
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

---

## Binary Records

The record structure could be JSON, CBOR (or BEVE or similar) or Binary.

|Encoding|Human Readable|Extensible|Parse Speed|Copy to Sign|Size     |
|--------|--------------|----------|-----------|------------|---------|
|JSON    | **yes**      | **yes**  | slow      | copy       | largest |
|CBOR    | no           | **yes**  | **fast**  | Maybe      | **medium** |
|Binary  | no           | no       | **NONE**  | **NO**     | **SMALLEST** |

Except for human readability, CBOR is better than JSON in cases where extensibility
is needed.

Other than that, binary is better in all respects.

We argue that extensibility of the main record is sufficiently done via record kinds,
tags and content, and therefore extensibility is not needed for this structure.

We sacrifice:

* Human readability
* Extensibility

We gain:

* Zero parsing required
* Zero copying for signing required
* Smallest size has been achieved

BTW: JSON also has character encoding ambiguities.

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

## Client-Server

Mosaic is NOT peer-to-peer. It turns out that peer-to-peer is difficult because most computers are not
fully connected to the Internet. And as there is nothing particular difficult in running a server that
*is* fully connected to the Internet (given VPS availability), being strictly peer-to-peer doesn't seem
advantageous. So we choose the more rock-solid client-server architecture.

However, we opt for a dumb-server smart-client design to put more control into the hands of the users.

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

Users need to keep their secret key secure so they don't have their identity stolen. But
they also need to access their secret key online in order to use the protocol. By splitting
the keys into a master key and multiple device keys (or subkeys) we can enable very secure
master key storage and usage without impeding the ability to operate online with subkeys.

---

## No IP Privacy

Several IP privacy network layers exist which already do a good job (Tor, i2p, VPNs). The IP privacy
issue is almost entirely orthogonal to the application protocol, and so architecturally it makes sense
to separate application layers from privacy layers. There is no good reason to reinvent another privacy
layer just for Mosaic since Mosaic can run on top of an already existing general-purpose privacy layer.

---

## QUIC

HTTP has a lot of baggage. WebSockets layers on top of that.

What we really need is size-unrestricted datagrams within TLS.  QUIC
gives us that if we do our own framing within a QUIC stream.

QUIC is widely deployed due to its use in HTTP 1.3. QUIC is also much
faster due to fewer roundtrips and no head-of-line blocking, while
taking care of TLS, multiplexed streams, congestion control and other
details that we would not dare to reimplement.

The only concern would be if the port is not opened, and in this case
a server can run on UDP port 443 (even though it is not speaking HTTP).

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

We provided separate flags for application-specific usage. These are quicker to look
up than tag-based or content-based flags.

### Duplicate Timestamp

We have the timestamp twice. We had to duplicate it because the big-endian
timestamp in the ID is not hashed or signed.

### Tags

Tag values should not be too large as they need to be indexed by relays.

Constraining the value to 253 bytes allows an entire TLV (with 16-bit type and 8-bit length) to
fit within 256 bytes.

While for some tag types a length could be inferred, this is not true in general. Applications
should not be required to recognize every tag type to look up its known length or type-specific
method of length calculation.

Some tag types start with padding in the value in order to better align
their data.

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

The zeroes in the ID maintain 64-bit alignment.

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

Additionally, servers can now present themselves at multiple transport endpoints.

By having a strong form of identification, clients can maintain reliable reputational
information about servers.

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
have not gotten yet. They can use this received-at data to avoid missing events
that were timestamped earlier but arrived later than the last time they checked for
events.

---

## Timestamps

### Leap Seconds

UTC is a discontinous time scale that is occasionally adjusted by leap seconds.
Unixtime is derived from UTC and is thus also discontinuous.  Subtracting two
unixtimes could give a time interval that is off by up to 28 seconds (for
example when comparing dates before 1 Jan 1972 with today).

### Milliseconds

Millisecond unixtimes only take 6 bytes and in 47 bits give us more than
4000 years before they roll over.

### Bit 48

The most significant bit of timestamps is set to 0 in case software interprets
it as a signed integer, to preserve sorting.

---

## TLS

We use TLS security because it is heavily researched and state-of-the-art.

But we don't use the X.509 certificates as intended since both client and server are known by
their public keys, not by DNS names.  Instead we use RawPublicKey or self-signed certificates.

Clients are authenticated via client-side certificates.
