# Protocol

Clients always contact servers. Therefore the initial message always comes from a client.

Specific encodings depend on the transport. Refer to the transport for details:

* [QUIC](quic.md)
* [TCP](tcp.md)
* [WebSockets](websockets.md)

## HELLO

The HELLO message includes the following information:

* The major protocol versions supported by the client
* The application features supported by and requested by the client
* A nonce for the server to sign as part of server authentication
* Whether the client wishes to authenticate, and if so, the client user's public key.

## HELLO ACK

In response to the HELLO information, the server next responds with HELLO ACK information
which includes the following information:

* The major protocol version to use
* The application features to use
* The nonce in HELLO signed by the server keypair
* If the client requested to authenticate, a nonce to challenge the client. In this case,
  the HELLO AUTH message is required from the client next.
* An optional service URL presented to clients.
  Servers MAY present a service URL for a website which users can visit in
  order to manage their relationship with the server (e.g. sign up for an
  account, make payment, view logs, or anything else that is relevant to
  that relationship). These kinds of activities are not standardized here.

## HELLO AUTH

In case a HELLO AUTH is needed to complete client authentication, it includes the
following information:

* The nonce in HELLO ACK signed by the client keypair

## Subsequent messages

After this point, all further communication is client-initiated. See [Messages](messages.md)
for details.
