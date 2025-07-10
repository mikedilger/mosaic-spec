# Mosaic over Websockets

<status>PAGE STATUS: Incomplete</status>

## When to choose WebSockets

When a server wishes to be available to web-based clients, it must present at least
one endpoint over WebSockets as this is the only transport that web-based clients
can reliably access.

## WebSockets Reference

See the [WebSockets](https://datatracker.ietf.org/doc/html/rfc6455) RFC.

## TLS

Over WebSockets, Mosaic MUST use TLS version at least 1.2, preferably
[TLS 1.3](https://datatracker.ietf.org/doc/html/rfc8446).

TLS certificates must be issued by Certificate Authorites commonly trusted
by browsers. This is so that web-based clients can access the server.

Client-side certificates are not used.

## Binary

All messages (outside of HELLO messages described below) use websockets binary.

Messages are formed as the binary message protocol specified in
[Protocol messages](messages.md). Each message starts with a single byte indicating
the type, followed by the data that such type requires.

Clients MUST only send client messages. If a server reads a server message
from a client, it SHOULD disconnect.

Servers MUST only send server messages. If a client reads a client message
from a server, it SHOULD disconnect.

## HELLO Messages

`HELLO`, `HELLO ACK` and `HELLO AUTH` are handled over the WebSockets transport in the following fashion:

### HELLO

HELLO information is encoded into HTTP headers within the upgrade request.

The `Sec-WebSocket-Protocol` header MUST be present and set to `mosaic2025`.
If this is not presented, a server may either refuse service and close the connection,
or presume the connection is nostr (if it is dual-stack). It SHOULD NOT
presume the connection is Mosaic.

`X-Mosaic-Versions` is a comma-separated list of major version numbers supported by the client.

`X-Mosaic-Features` is a comma-separated list of application features suppored by and requested
by the client.

`X-Mosaic-Authenticate-As` is a [mopub0](human_encodings.md) encoded public key of the client's user, only if the client wishes to authenticate.

`X-Mosaic-Server-Authenticate-Nonce` is a ASCII nonce string.

### HELLO ACK

The `Sec-WebSocket-Protocol` header MUST be present and set to `mosaic2025`.
If this is not presented, a client may either refuse service and close the connection,
or presume the connection is nostr (if it is dual-stack). It SHOULD NOT
presume the connection is Mosaic.

HELLO ACK information is also encoded into HTTP headers within the upgrade response.

`X-Mosaic-Version` is the single major version number decided by the server. It MUST be present in the `X-Mosaic-Versions` from the HELLO message.

`X-Mosaic-Features-Accepted` is a comma-separated list of application features accepted by the server.

`X-Mosaic-Server-Authentication` is a base64 encoded ed25519 signature of the nonce provided in the HELLO messag `X-Mosaic-Server-Authenticate-Nonce` header.

`X-Mosaic-Client-Authenticate-Nonce` is a server-generated nonce for the client to authenticate with, only in the case that the HELLO message contained a `X-Mosaic-Authenticate-As` header.

`X-Mosaic-Service-Url` is an optional service URL that servers may present to
clients.

### HELLO AUTH

This MUST is sent by the client as the first WebSocket binary message. See [Messages](messages.md).
