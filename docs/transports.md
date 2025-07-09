# Transports

The following transports are defined:

| Transport | Requirement |
|-----------|-------------|
| [QUIC](quic.md)   | Required [<sup>rat</sup>](rationale.md#quic) |
| [TCP with TLS 1.3](tcp.md) | Optional |
| [WebSockets over https with TLS 1.3](websockets.md) | Required [<sup>rat</sup>](rationale.md#websockets) |

## Transport Requirements

For compatibility purposes, all Mosaic clients and servers MUST be able to communicate over
the following universally supported transports:

* QUIC
* Websockets over HTTPS with TLS 1.3

Any server providing access via an alternate transport MUST also advertise an
endpoint under at least one universally supported transport.
