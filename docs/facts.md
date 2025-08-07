# Technical Facts

If you are seeking a quick answer to a technical question about Mosaic, this section may
help you.

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

Whenever multiple options are available (such as transport), a core
subset will be specified as required for compatibility.

Mosaic uses a
<t>client-server</t> [<sup>rat</sup>](rationale.md#client-server)
protocol architecture.
Records are simply stored and retrieved on select servers, there is no routing.
Mosaic is fully capable of peer-to-peer deployments by having servers optionally
run by peers (e.g. self-hosting).

Mosaic does not provide
<t>IP privacy</t> [<sup>rat</sup>](rationale.md#no-ip-privacy)
because this is an orthogonal concern that is easier to manage if it is decoupled from
Mosaic and provided by a lower network layer, such as a VPN or Tor.
But Mosaic should not interfere with IP privacy solutions.

Mosaic runs over any
<t>duplex communication</t> [<sup>rat</sup>](rationale.md#duplex-communication)
transport protocol that is
<t>TLS</t> [<sup>rat</sup>](rationale.md#tls)
secured such as <t>QUIC</t> [<sup>rat</sup>](rationale.md#quic) or TLS over TCP,
but requires WebSockets at a minimum for interoperability.

Mosaic users generate their own identities as digital signature cryptosystem
keypairs and Mosaic uses a
[master-key subkey](identity.md#master-keys-and-subkeys)
[<sup>rat</sup>](rationale.md#master-key-subkey)
design.

Mosaic [servers](identity.md#server-identities) have
<t>EdDSA ed25519</t> [<sup>rat</sup>](rationale.md#eddsa-ed25519)
key-based
<t>server identites</t> [<sup>rat</sup>](rationale.md#server-identities).

Mosaic identity and endpoint information is [bootstrapped](bootstrap.md) from
<t>Mainline DHT</t> [<sup>rat</sup>](rationale.md#mainline-dht).

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

The name Mosaic was not chosen for any particular reason. It is just a name, easy to
remember and pronounce. A throwback to the old NCSA Mosaic browser I suppose. It is
not an acronym. We always capitalize it even in the middle of a sentence.
