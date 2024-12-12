# Bootstrap

<status>PAGE STATUS: draft</status>

When you first find out about a new public key, you may already know by the
context if it represents a user or a server. But sometimes you don't even
know that.  You also may not know what servers this key uses to host it's
[key schedule](keyschedule.md) and [profile](profile.md) information, or which
it uses to publish it's records or receive messages.

Bootstraps are public digitally signed records designed to let you acquire
this kind of information.

We store bootstraps in Mainline DHT.

Bootstraps are not [records](record.md). They have their own format.

## Mainline DHT

We use <t>Mainline DHT</t> [<sup>rat</sup>](rationale.md#mainline-dht)
to store mutable data signed under an ed25519 signature
according to [BitTorrent BEP 0044](https://www.bittorrent.org/beps/bep_0044.html).

Limitations:

* Only 1000 bytes can be reliably stored, and some will be used for *bencoding*
  overhead and the salt, leaving us only 983 bytes of usable data.
* Data must be refreshed periodically otherwise it may be removed after a time.
  Users are responsible for refreshing data in the Mainline DHT which will
  disappear over time. Mechanisms for this are out of scope for Mosaic Core.
* Data storage and retrieval may take a few seconds, and should not be done too
  frequently. Software SHOULD cache results for at least 2 hours.
* You need a bootstrap to get started and find peers. `router.utorrent.com`
  and `router.bittorrent.com` are common but could be targetted in an attack.
  However, you can find many more or setup your own
  [bootstrap server](https://github.com/bittorrent/bootstrap-dht).

### Salt

We use a <t>salt</t> [<sup>rat</sup>](rationale.md#salt)  of `msb24`
for server bootstraps and `mub24` for user bootstraps.

### Sequence Numbers

<t>Sequence numbers</t> [<sup>rat</sup>](rationale.md#sequence-numbers)
should start at 1 and monotonically increase with each write.

### Rust code

There is a rust library to access this called [mainline](https://github.com/pubky/mainline)

## Bootstrap Format

Bootstraps (the data after the bencoded prefix) are UTF-8 valid text up
to 983 bytes long, and consist of a series of lines separated with a single
ASCII Line Feed (LF) character (`\n`). Lines MUST not have trailing whitespace.

Two kinds of bootstraps may be stored, based on whether the identity
represents a server or a user.


## Server Bootstraps

Server bootstraps specify the Internet locations (protocol, host and
port) that the server is available at.

A server bootstrap starts with the line `S`.

Each subsequent line in a server bootstrap specifies a URL where
the server can be accessed.

These URLs MUST contain a scheme and a host, may optionally contain a port, and must
specify the root path (`/`).  They must NOT contain user, password, query, or fragment
section. If any of those is found in one of these URL, software MUST prune such
information.

Only secure transports with TLS are defined. TLS must be version 1.2 or 1.3.
The only known schemes currently are `wss` and `https` (`https` being
designated for [WebTransport](webtransport.md))

The order of the entries expresses the preference or priority. Earlier
entries are preferred by the server to later ones.

Here are some examples of server bootstrap lines:

`wss://203.0.113.1` specifies WebSockets with TLS on port 443 at IP address
`203.0.113.1`.

`wss://203.0.113.0:5198` specifies WebSockets with TLS on port 5198 at IP
address `203.0.113.0`.

`wss://myserverlk23lkjsefo8u.onion` specifies WebSockets with TLS on port 443
over Tor to onion site `myserverlk23lkjsefo8u.onion`.

`wss://[2001::130F::09C0:876A:130B]` specifies WebSockets with TLS on port 443
over IPv6 to address `2001::130F::09C0:876A:130B`.

`https://mosaic.example:555` specifies WebTransport with TLS on port 555 to DNS
node mosaic.example (https is to be interpreted as WebTransport, not REST).

Servers are expected to operate as their own inbox/outbox and encryption
server. So they do not require the same data as the user bootstrap.

## User Bootstrap

User bootstraps specify servers that the user uses, and how the user uses them.

The order of the entries expresses the preference or priority. Earlier entries are
preferred by the user to later ones.

A user bootstrap starts with the line `U`.

Each line consists of two parts separated by a single ASCII space.

### Server Usage Character

This first part is a single character that encodes that kind of usage.

There are three defined server usages:

* **Outbox** - Outbox servers are where users publish public records meant to
be read by anyone who is following the person's public content.
The [key schedule](keyschedule.md) and [profile](profile.md) are published
here.

* **Inbox** - Inbox servers are where users receive records that reference them,
and where other users can follow replies to messages created by them.

* **Encryption** - Encryption servers function like an inbox but handle
private encrypted messages (defined outside of Mosaic core) that only the
user can read back.

Outbox is indicated by bit 0 (`1<<0`) in the character. A 1 bit means the
server is an outbox server.

Inbox is indicated by bit 1 (`1<<1`) in the character. A 1 bit means the server
is an inbox server.

Encryption is indicated by bit 2 (`1<<2`) in the character. A 1 bit means the
server is an encryption server.

Bits 5 and 6 are always on. This is an ASCII '0' (48, 0x30). However a `0`
should never be used as a server usage character as this would indicate no
server usages, which is invalid as such a line should not exist.

For example, to indicate only outbox usage, use character `1`. To indicate all
three usages, use `7`.

Conveniently with this encoding the ASCII number also matches the relavant bits.

### Server Key

The second part is the server's public key, encoded using base64 (using the
standard alphabet of RFC 4648). This will be 44 characters long ending in an
`=` symbol.

### Example

Here is an example user bootstrap:

```
U
1 FB42YsY/CV2FqlMrI4CNeaZ2LnCHXzXmmdGKA+UsuBc=
3 GQ859t+vK9gfYolOMfGB0VD/+kjk3iGFjxHj0GfhMos=
2 L+RDYOrIKID+eEK81510TJ1pQOQW7kMrA10MwKOu0Iw=
3 uBpfOVe3ooWMnc1RdMbYKBAIcHlfl2FsQU67lK2CJ8A=
6 VKLdex3KykACzM0JpRfduelqwytel1AZGaXuv4sZhfU=
```

Based on size limits of 983 bytes, no more than 20 server entries can be
listed (but see below for other limitations on the number of servers).

Should servers become unreliable, users can change servers and update these
bootstrap entries at any time.

### Usage of servers and limits on their number

*Maximums*: Users SHOULD list no more than 4 redundant servers of any kind,
since more redundancy provides strongly diminishing benefit at a linearly
increasing network traffic cost.  Software MUST utilize the first four
servers of the appropriate kind listed, and MAY tolerate additional servers
but optionally MAY ignore additional servers.

*Minimums*: Users SHOULD have at least one outbox and at least one inbox.
Users MAY have no encryption servers but they will not be able to receive
encrypted messages.
