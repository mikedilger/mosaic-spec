# URL

<status>PAGE STATUS: early draft</status>

Mosaic servers are located using URLs. Mosaic server URLs are subject to the
following rules.

* Mosaic URLs MUST contain one of the following **scheme**s:
    * `mosaic` for Mosaic over [QUIC](quic.md)
    * `wss` for Mosaic over [WebSockets](websockets.md)
* Mosaic URLs MUST contain a **host**. This SHOULD normally be an IP address (either
  IPv4 or IPv6) but it MAY be a DNS name. DNS names are useful primarily when
  using Tor (.onion) or I2P (.i2p), but regular DNS may also be used although this
  is recommended against since DNS can be censored.
* Mosaic URLs MAY contain a **port**. If a port is not provided, then the following
  defaults apply
    * The `mosaic` default port is 1320.
    * The `wss` default port is 443.
* Mosaic URLs MUST specify the root path `/`.
* Mosaic URLs MUST not contain a user, password, query, or fragment section. If any
  of these is found, software MUST prune such information.
