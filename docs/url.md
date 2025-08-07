# URL

Mosaic servers are located using URLs. Mosaic server URLs are subject to the
following rules.

## Scheme

Mosaic URLs MUST contain one of the following **scheme**s which indicate which
transport is used:

* `mosaic` for Mosaic over [QUIC](quic.md)
* `mosaictcp` for Mosaic over [TCP with TLS](tcp.md)
* `mosaicwss` for Mosaic over [WebSockets](websockets.md)

## Host

Mosaic URLs MUST contain a **host**.

This SHOULD normally be an IP address (either IPv4 or IPv6) but it MAY be a DNS name.

Hosts may also be Tor onion sites (specified as a DNS name). When hosts are exposed
through Tor, the scheme MUST NOT be `mosaic` since QUIC cannot transit Tor.

## Port

Mosaic URLs MAY contain a **port**.

If a port is not provided, then the following defaults apply

* For `mosaic` the default port is 1320.
* For `mosaictcp` the default port is 1320.
* For `mosaicwss` the default port is 443.

## Path

Mosaic URLs MUST specify the root path `/` exactly and no other path.

## Other URL components

Mosaic URLs MUST NOT contain a user, password, query, or fragment section.

If any of these is found, software MUST ignore and MAY prune such information.
