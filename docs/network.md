# Network Architecture

## Clients

Mosaic clients are meant to serve users, and thus sit on user-operated notes, often
phones or desktops or laptops. They may be behind a NAT gateways, or indeed multiple
layers of NAT gateways. Mosaic clients initiate connections to servers, and never the
reverse.

## Servers

Mosaic servers are meant to remain online and reliable, and SHOULD be accessible from
the broader Internet.

Servers not accessible from the broader Internet, either because they are behind NAT
gateways, or because they operate from a Tor onion site domain, are NOT suitable for
<t>inbox</t>, <t>outbox</t> or <t>encryption</t> roles, but they MAY serve other
applications.

## Server Roles

Servers fulfilling certain kinds of roles must meet the requirements of those roles.
Servers can (and often do) serve many or even all server roles.

### Outbox

Outbox servers are where users _publish_ public records meant to be read by anyone who wishes to.

Servers fulfilling an Outbox role are subject to the following:

* They MUST be broadly accessible from the Internet, not behind NAT, and not running
  from a Tor onion site.
* They SHOULD have a high level of uptime.
* They MUST accept messages from the users that they are serving, except where abuse prevention
  is concerned.
* They MUST deliver such messages to anybody who requests them without requiring
  authentication and without requiring payment.

### Inbox

Inbox servers are where users _receive_ records that reference them, and where other users can
follow replies to the user's messages which are posted here by other people.

Servers fulfilling an Inbox role are subject to the following:

* They MUST be broadly accessible from the Internet, not behind NAT, and not running
  from a Tor onion site.
* They SHOULD have a high level of uptime.
* They MUST accept messages from strangers, except where abuse prevention is concerned,
  without requiring authentication and without requiring payment.
* They MUST serve messages to the intended recpients after requiring authentication.
* They SHOULD serve messages to anybody without requiring authentication and without requiring
  payment.
* They MAY withhold serving messages prior to abuse prevention and moderation.

### Encryption

Encryption servers function like _inbox_ servers but handle private encrypted messages (defined
outside of Mosaic core) that only the user can read back.

Servers fulfilling an Encryption role are subject to the following:

* They MUST be broadly accessible from the Internet, not behind NAT, and not running
  from a Tor onion site.
* They SHOULD have a high level of uptime.
* They MUST accept messages from strangers, except where abuse prevention is concerned,
  without requiring authentication and without requiring payment.
* They MUST serve messages to the intended recpients after requiring authentication.
* They MUST NOT deliver messages to anybody other than the indended recipient.
