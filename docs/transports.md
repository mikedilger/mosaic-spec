# Transports

The following transports are defined:

| Transport | Requirement |
|-----------|-------------|
| [QUIC](quic.md)   | Required [<sup>rat</sup>](rationale.md#quic) |
| [TCP with TLS 1.3](tcp.md) | Required |
| [WebSockets over https with TLS 1.3](websockets.md) | Optional |

## Transport Requirements

For compatibility purposes, all Mosaic clients and servers MUST be able to communicate over
the following universally supported transports:

* QUIC
* TCP with TLS 1.3

Any server providing access via an alternate transport MUST also advertise an
endpoint under at least one universally supported transport.
