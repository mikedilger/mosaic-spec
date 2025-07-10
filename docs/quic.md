# Mosaic over QUIC

## When to choose QUIC

The QUIC transport is the highest performance option.
Most significantly, QUIC has fewer roundtrips by design.
But it also head-of-line blocking, has optional out-of-order delivery (Mosaic does
not utilize this), and supports multiplexing. It also maintains connections when
clients switch networks, for example when mobile phones switch from WiFi to 4G.

QUIC cannot be used by browser-based clients. Browser-based clients must use the
[WebSockets](websockets.md) transport.

QUIC cannot be used over Tor. Privacy-conscious users may prefer to use the
[TCP](tcp.md) transport over Tor.

## QUIC reference

See the [QUIC](https://datatracker.ietf.org/doc/html/rfc/rfc9000) RFC.

## TLS

Over QUIC, Mosaic MUST use TLS version at least 1.3 as required by QUIC.

TLS MUST specify the EdDSA signature algorithm, using the ed25519 signing
keys.

TLS certificates SHALL be either RawPublicKey or self-signed, and use
the ed25519 public key (either the server's master public key or the client's
signing public key).

Servers SHOULD request client-side certificates if they wish to authenticate
users. If clients do not provide certificates, they are to be considered
anonymous. Servers MAY present different services depending on whether a user is
authenticated or not.

The self-signed certificate distinguished name hostname MUST be "mosaic".

Server-side certificates are REQUIRED and authenticate the server to the client.
These MUST use the ED25519 signature scheme and certificate verifier.
The certificate verifier MUST accept self-signed certificates if they are validly
signed by the public key within, and interpret that public key to be the Mosaic
key of the server.

Client-side certificates are OPTIONAL and authenticate the client to the server.
These MUST use the ED25519 signature scheme and certificate verifier.
The certificate verifier MUST accept self-signed certificates if they are validly
signed by the public key within, and interpret that public key to be the Mosaic
key of the user.

## QUIC ALPN

The QUIC ALPN used by Mosaic is "mosaic".

## Streaming and Framing

Clients MAY open multiple streams and write protocol messages into them, however Mosaic
works just fine over a single stream.

Servers MUST handle multiple streams, and reply to protocol messages on the same stream
from which the request came down.

Messages are framed based on their prefix of 1 type byte and 3 length bytes.

## HELLO Messages

`HELLO`, `HELLO ACK` and `HELLO AUTH` are handled over the QUIC transport as normal
[Messages](messages.md). [<sup>rat</sup>](rationale.md#0-rtt)
