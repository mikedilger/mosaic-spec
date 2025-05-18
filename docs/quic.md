# Mosaic over QUIC

<status>PAGE STATUS: Early Draft</status>

Mosaic benefits from TLS, but does not benefit from much else that HTTP or WebSockets
provide other than framing. But framing is quite easy.

QUIC brings a lot of performance and usability benefits such as avoiding head-of-line
blocking, fewer roundtrips by design, optional out-of-order delivery, and multiplexing.

TLS must be version 1.3 only.

## Framing

We cannot use QUIC datagrams due to unreliability and size limitations. So we open a single bidirectional stream for Mosaic and manage our own framing.

Framing is simple:

* Eight byte length of mosaic message
* The mosaic message itself

Some mosaic messages are already framed, but we do this outer framing because they are not all uniformly framed.
