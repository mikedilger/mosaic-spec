# Transport Requirements

For compatibility purposes, all Mosaic clients and servers MUST be able to
communicate over the following universally supported transports:

* QUIC
* TCP secured with TLS 1.3

Other transports (such as WebSockets) are OPTIONAL.

Any server providing access via an alternate transport MUST also advertise an
endpoint under at least one universally supported transport.
