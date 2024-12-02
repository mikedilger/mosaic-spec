# Bootstrap

<status>PAGE STATUS: early draft</status>

When you first find out about a new public key, you may already know if it represents a
user or a server (by the context) or you may not even know that.  And you also may not
know what servers this key uses to host it's [key schedule](keyschedule.md),
and [profile](profile.md) information, or to publish it's records or receive messages.

We store bootstrap records in Mainline DHT

## Mainline DHT

We use Mainline DHT to store mutable data signed under an ed25519 signature
according to [BEP 0044](https://www.bittorrent.org/beps/bep_0044.html).

Rationale:

* Mainline DHT is distributed and censorship resistant, including being resistant to
  (or able to detect) Sybil attacks. It also has this mutable data functionality and
  works with ed25519 signed data.

Limitations:

* Only 1000 bytes can be reliably stored, and some will be used for bencoding overhead
  and the salt, leaving us only 983 bytes of usable data.
* Data must be refreshed periodically otherwise it may be removed after a time.
  Users are responsible for refreshing data in the Mainline DHT which will disappear over time.
  Mechanisms for this are out of scope for Mosaic Core.
* Data storage and retrieval may take a few seconds, and should not be done too frequently.
  Software SHOULD cache results for at least 2 hours.

### Salt

We use a salt of "msb24" for server bootstrap records and "mub24" for user bootstrap records.

Rationale:

* We use the salt to avoid collisions, in case the same ed25519 identity keypair is used
  by both mosaic and [pubky](https://github.com/pubky), or in case we need to change the
  format of the bootstrap record in the future, and because we have two different kinds of
  bootstrap records already (this allows a server keypair to also be a user keypair without
  collision).
* This salts are short enough to not use too much space.

### Sequence Numbers

Sequence numbers should start at 1 and monotonically increase with each write.

### Rust code

A rust library to access this:  https://github.com/pubky/mainline

## Bootstrap Record Format

Bootstrap records (the data after the bencoded prefix) are UTF-8 valid text up to
983 bytes long, and consist of a series of lines separated with a single ASCII
Line Feed (LF) character (0x0A, \n). Lines MUST not have trailing whitespace.

Two kinds of records may be stored, based on whether the identity represents a server
or a user.


## Server Bootstrap Records

Server bootstrap records specify the internet locations (protocol, host and port) that the server
is available at.

A server bootstrap record starts with the line `S`.

Each subsequent line in a server bootstrap record specifies a URL where the server can be
accessed. There can be any number of lines. However, the total length of the data cannot
exceed 983 bytes.

URLs must contain scheme and host, and may optionally contain a port.  URLs must NOT contain
user, password, path, query, or fragment sections.  If any of those is found in a URL,
software MUST prune such information. This includes pruning trailing slashes (which are paths).

Only secure transports with TLS are defined. TLS must be version 1.2 or 1.3. The only known
schemes currently are `wss` and `https`.

Here are some examples of server bootstrap record lines:

`wss://203.0.113.1` specifies WebSockets with TLS on port 80 at IP address 203.0.113.0.

`wss://203.0.113.0:5198` specifies WebSockets with TLS on port 5198 at IP address 203.0.113.0.

`wss://myserverlk23lkjsefo8u.onion` specifies WebSockets with TLS on port 443 over Tor.

`wss://[2001::130F::09C0:876A:130B]` specifies WebSockets with TLS over IPv6 on port 443.

`https://mosaic.example:555` specifies WebTransport with TLS on port 555 to DNS node mosaic.example
(https is to be interpreted as WebTransport, not REST).

Servers are expected to operate as their own inbox/outbox and encryption server. So they do not
require the same records as the user bootstrap records.


## User Bootstrap Records

User bootstrap records specify servers that the user uses, and how the user uses them.

A user bootstrap record starts with the line `U`.

Each line consists of two parts

### Server Usage Character

This first part is a single character that encodes that kind of usage.

There are three defined server usages:

* **Outbox** - Outbox servers are where users publish public
records meant to be read by anyone.  Bootstrap records such as the [key schedule](keyschedule.md) and
[profile](profile.md) are published here.

* **Inbox** - Inbox servers are where users check for records
that reference them, so that they can be contacted.

* **Encryption** - Encryption servers function like an
inbox but handle private encrypted messages (defined outside of Mosaic core).

Outbox is indicated by bit 0 (1<<0) in the character. A 1 bit means the server is an outbox server.

Inbox is indicated by bit 1 (1<<1) in the character. A 1 bit means the server is an inbox server.

Encryption is indicated by bit 2 (1<<2) in the character. A 1 bit means the server is an encryption server.

Bits 5 and 6 are always on. This is an ASCII '0' (48, 0x30). However a '0' should never be used, as this
would indicate no server usages.

For example, to indicate only outbox usage, use character '1'. To indicate all three usages, use '7'.

Conveniently with this encoding the ASCII number also matches the relavant bits.

### Server Key

The second part is the server's public key, encoded using base64 (using the standard alphabet of RFC 4648).
This will be 44 characters long ending in an '=' symbol.

### Example

Here is an example user bootstrap record:

```
U
1FB42YsY/CV2FqlMrI4CNeaZ2LnCHXzXmmdGKA+UsuBc=
3GQ859t+vK9gfYolOMfGB0VD/+kjk3iGFjxHj0GfhMos=
2L+RDYOrIKID+eEK81510TJ1pQOQW7kMrA10MwKOu0Iw=
3uBpfOVe3ooWMnc1RdMbYKBAIcHlfl2FsQU67lK2CJ8A=
6VKLdex3KykACzM0JpRfduelqwytel1AZGaXuv4sZhfU=
```

Based on size limits of 983 bytes, no more than 20 records can be stored (but see below for other
limitations on the number of servers).

Should servers become unreliable, users can change servers and update these records at any time.

### Usage of servers and limits on their number

*Maximums*: Users SHOULD list no more than 4 redundant servers of any kind, since more redundancy
provides strongly dimishing benefit at a linearly increasing network traffic cost.  Software MUST
utilize the first four servers of the appropriate kind listed, and MAY tolerate additional servers
but optionally MAY ignore additional servers.

*Minimums*: Users SHOULD have at least one outbox and at least one inbox. Users MAY have no
encryption servers but they will not be able to receive encrypted messages.
