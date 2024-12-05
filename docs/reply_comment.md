# Reply Comment

<status>PAGE STATUS: incomplete</status>

Kind = 0x5 - Reply Comment

## Flags

* 0x01 ZSTD - may be on or off
* 0x02 FROMAUTHOR - may be on or off to control distribution.
* 0x04 TORECIPIENTS - may be on or off.
* 0x08 NOBRIDGE - may be on or off
* 0x10 EPHEMERAL- MUST be off. Reply comments are not ephemeral.

## Tags

This MUST include exactly one reply tag, either 0x2 or 0x3.

This MUST include exactly one root tag, either 0x2 or 0x3.

## Server Used

These are posted to all of the author's Outbox servers and also to all
of the parent Record author's Inbox servers.
