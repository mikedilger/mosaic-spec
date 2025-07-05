# Mosaic

<status>PAGE STATUS: early draft</status>

Mosaic is a *work in progress*.  This specification is incomplete and an early draft.

When you see a <t>superscript</t> [<sup>rat</sup>](rationale.md), the "rat" is a link
to the rationale behind this. Rationale is kept so we can remember why decisions were
made, but kept separate to not clutter the specification.

## Introduction

Mosaic is a
<t>distributed</t> [<sup>rat</sup>](rationale.md#distributed)
and
<t>sovereign</t> [<sup>rat</sup>](rationale.md#sovereign)
social application base-layer specification for the Internet.

Mosaic is meant to be an underlying unopinionated layer.
Your opinionated social media application rides on top.
But we have to make some decisions at this layer, so we can't be entirely unopinionated.

[Applications](applications.md) are specified separately, but register an application
ID with Mosaic (or they just choose one randomly). Two applications are defined within
this spec: <t>Mosaic Core</t> and <t>Mosaic Social Media</t>.

Whenever multiple options are available (such as transport), a core subset will be
specified as required for compatibility.

Mosaic uses a
<t>client-server</t> [<sup>rat</sup>](rationale.md#client-server)
architecture.

Mosaic runs over any
<t>duplex communication</t> [<sup>rat</sup>](rationale.md#duplex-communication)
transport protocol that is
<t>TLS</t> [<sup>rat</sup>](rationale.md#tls)
secured such as <t>QUIC</t> [<sup>rat</sup>](rationale.md#quic) or TLS over TCP.
Mosaic does not need nor utilize the functionality of HTTP.

Mosaic users generate their own identities as
<t>EdDSA ed25519</t> [<sup>rat</sup>](rationale.md#eddsa-ed25519)
keypairs and Mosaic uses a
[master-key subkey](identity.md#master-keys-and-subkeys)
[<sup>rat</sup>](rationale.md#master-key-subkey)
design.

Mosaic [servers](identity.md#users-versus-servers) have key-based
<t>server identites</t> [<sup>rat</sup>](rationale.md#server-identities).

Mosaic identity and endpoint information is [bootstrapped](bootstrap.md) from
<t>Mainline DHT</t> [<sup>rat</sup>](rationale.md#mainline-dht)

Mosaic does not provide
<t>IP privacy</t> [<sup>rat</sup>](rationale.md#no-ip-privacy)
because this is an orthogonal concern that is easier to manage if it is decoupled from
Mosaic and provided by a lower network layer, such as a VPN or Tor.
But Mosaic should not interfere with IP privacy solutions.

Mosaic [records](record.md) are encoded in a
<t>binary</t> [<sup>rat</sup>](rationale.md#binary-records) format within the
protocol messages themselves.
So are [filters](filter.md) and [protocol messages](messages.md).

Mosaic [records](record.md) are [editable](reference.md)
if the application layer wishes them to be, as all records have (and can be addressed by)
a [reusable address](reference.md#address-reference)
as well as a unique [hash-based id](reference.md#id-reference).

[Timestamps](timestamps.md) [<sup>rat</sup>](rationale.md#timestamps)
account for <t>leap seconds</t>, have <t>nanosecond accuracy</t>, and extend
out to year 2262.

Clients and Servers
<t>remember the time that records are received</t>
[<sup>rat</sup>](rationale.md#storing-received-at-timestamps).

Mosaic is an offshoot of [nostr](https://github.com/nostr-protocol).

Mosaic uses the <t>BLAKE3</t> [<sup>rat</sup>](rationale.md#blake3) hashing function.

### The name Mosaic

No reason. Just a name. Easy to remember and pronounce. A throwback to
the old NCSA Mosaic browser I suppose. It is not an acronym. We always
capitalize it even in the middle of a sentence.

### MUST, SHOULD, and MAY

The capitalized words "MUST", "REQUIRED", "SHALL", "MUST NOT" and
"SHALL NOT" indicate an absolute requirement in order to be in compliance with this
specification, and violation means the software does not comply with the Mosaic
specification.

The capitalized words "SHOULD", "SHOULD NOT", and "RECOMMENDED" indicate
strong advice for the default case, but there may be valid exceptions and software
that does otherwise is not in violation of the specification.

The capitalized words "MAY" and "OPTIONAL" indicate a choice that is truly
optional.

These definitions are differently worded, but are not meant to be functionally different
from [rfc2119](https://datatracker.ietf.org/doc/html/rfc2119).

