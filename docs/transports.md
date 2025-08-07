# Transports

NOTICE: Once WebTransport matures further, we may swap it in for WebSockets because it
would be strictly better:

* QUIC, if available, with fallback to HTTP/2 (thus Tor can work)
* serverCertificateHashes (to avoid trust in DNS and CAs)

NOTICE: Development is proceeding with QUIC first.

Mosaic defines three transports with the following characteristics:

| Transport | Required | Browser clients | Tor support | Trust | Performance |
|-----------|----------|-----------------|-------------|-------|-------------|
| [WebSockets](websockets.md) | Required [<sup>rat</sup>](rationale.md#websockets) | Yes | Yes | Relies on CAs and DNS | Lowest |
| [QUIC](quic.md)   | Optional [<sup>rat</sup>](rationale.md#quic) | No | No | None | Highest |
| [TCP](tcp.md) | Optional | No | Yes | None | Medium-Low |

## Transport Requirements

For compatibility purposes, all Mosaic servers and clients MUST be able to communicate over
Websockets over HTTPS. Every Mosaic server MUST advertise at least one Websockets over HTTPS
endpoint. This requirement is in place to support browser-based clients, so that every
server can be contacted by every client.
