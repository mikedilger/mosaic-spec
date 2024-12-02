# Mosaic over Websockets

<status>PAGE STATUS: Draft</status>

## TLS

Over WebSockets, Mosaic must use TLS version at least 1.2, preferably
[TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446).

TLS should specify the EdDSA signature algorithm, using the ed25519 signing
keys.

TLS certificates shall be either RawPublicKey or self-signed, and use
the ed25519 public key (either the server's master public key or the client's
signing public key).

Servers SHOULD request client-side certificates if they wish to authenticate
users. If clients do not provide certificates, they should be considered
anonymous. Servers MAY present different services depending on whether a user is
authenticated or not.


## WebSockets

Mosaic messages are transported over
[WebSockets](https://datatracker.ietf.org/doc/html/rfc6455)

### Sec-WebSocket-Protocol

Clients must present the `Sec-WebSocket-Protocol` header field in the
handshake with WebSocket with the value `mosaic2024`. If this is not
presented, a server may either refuse service and close the connection,
or presume the connection is nostr (if it is dual-stack). It should not
presume the connection is Mosaic.

Servers MUST reply with the protocol they have accepted in the same
header, at this point being only 'mosaic2024'.

This subprotocol is not (as of this writing) registered with IANA, but does
not conflict with registered subprotocols.

### X-Mosaic-Features

Clients SHOULD NOT present an `X-Mosaic-Features` header in the handshake
because features have not currently been defined.

Servers MUST check for an `X-Mosaic-Features` header. If one is specified,
split it's contents on semicolons. Remove all features you are unable to
service. Join these back together and return an `X-Mosaic-Features` header
with this string. If that string was empty, use a `-` instead.

Clients MUST check for an `X-Mosaic-Features` header during negotiation,
and configure themselves to use only the features that the server returned
to them, considering `-` as the empty set.

## Binary

All messages use websockets binary.

Messages are formed as the binary message protocol specified in 
[protocol](protocol.md). Each message starts with a single byte indicating
the type, followed by the data that such type requires.

Clients MUST only send client messages. If a server reads a server message
from a client, it SHOULD disconnect.

Servers MUST only send server messages. If a client reads a client message
from a server, it SHOULD disconnect.
