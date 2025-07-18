# Mosaic over TCP

## When to choose TCP

TCP is the best transport in cases where a user wishes to have a high degree of privacy
and also a high degree of censorship resistance, because TCP can be used through Tor,
and also the TCP implementation does not rely on CAs or DNS.

However the TCP transport does not work in browser-based clients, in which case only
the [WebSockets](websockets.md) transport will suffice.
It is also not as high-performance as the [QUIC](quic.md) transport.

## TLS

Over TCP, Mosaic MUST use TLS version at least 1.2, preferably
[TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446).

TLS MUST specify the EdDSA signature algorithm, using the ed25519 signing
keys.

TLS certificates SHALL be either RawPublicKey or self-signed, and use
the ed25519 public key (either the server's master public key or the client's
signing public key).

Servers SHOULD request client-side certificates if they wish to authenticate
users. If clients do not provide certificates, they are to be considered
anonymous. Servers MAY present different services depending on whether a user is
authenticated or not.

## HELLO Messages

`HELLO`, `HELLO ACK` and `HELLO AUTH` are handled over the TCP transport as normal
[Messages](messages.md). [<sup>rat</sup>](rationale.md#0-rtt)
