# Mosaic over Websockets

<status>PAGE STATUS: Incomplete</status>

## TLS

WebSockets transport require TLS version at least 1.2 or preferably
[TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446)

TLS server-side certificates must be either self-signed using the server's signing key,
or preferably in RawPublicKey form.

Servers SHOULD request client-side certificates if they wish to authenticate users
(most do).

Clients MAY provide such certificates either self-signed or in RawPublicKey form,
presenting their signing key. If clients do not provide such, they should be considered
anonymous.

Servers MAY present different services depending on whether a user is authenticated or not.

TLS should specify the EdDSA signature algorithm, using the ed25519 signing keys.

## WebSockets

Mosaic messages are transported over [WebSockets](https://datatracker.ietf.org/doc/html/rfc6455)

Clients must present the `Sec-WebSocket-Protocol` header field in the handshake with
WebSocket with the value `mosaic2024`.  If this is not presented, a server may either
refuse service and close the connection, or presume the connection is nostr (if it is
dual-stack). It should not presume the connection is Mosaic.

Clients MAY present multiple additional protocol names (probably representing future
versions of Mosaic), and servers should accept whichever one they wish.

Servers MUST reply with the protocol they have accepted, at this point being only 'mosaic2024'.

This subprotocol is not (as of this writing) registered with IANA, but does not conflict
with registered subprotocols.

## Binary

All messages use websockets binary, and include the binary protocol message
literally as specified in [protocol](protocol.md)
