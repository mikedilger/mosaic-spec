# Mosaic over QUIC

<status>PAGE STATUS: Early Draft</status>

Mosaic benefits from TLS, but does not benefit from much else that HTTP or WebSockets
provide other than framing. But framing is quite easy.

QUIC brings a lot of performance and usability benefits such as avoiding head-of-line
blocking, fewer roundtrips by design, optional out-of-order delivery, and multiplexing.
It also maintains connections when clients switch networks, for example when mobile
phones switch from WiFi to 4G.

TLS MUST be version 1.3 only.

The QUIC ALPN used by Mosaic is "mosaic".

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

## Streaming and Framing

Clients MAY open multiple streams and write protocol messages into them, however Mosaic
works just fine over a single stream.

Servers MUST handle multiple streams, and reply to protocol messages on the same stream
from which the request came down.

Messages are framed based on their prefix of 1 type byte and 3 length bytes.
