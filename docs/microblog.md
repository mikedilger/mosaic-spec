# Microblog

<status>PAGE STATUS: incomplete</status>

Kind = 0x4 - Root

## Payload

The payload follows the [Human Readable Content](human_readable_content.md) rules.

## Flags

* 0x01 ZSTD - may be on or off
* 0x02 FROMAUTHOR - may be on or off to control distribution.
* 0x04 TORECIPIENTS - MUST be off. Microblogs are public.
* 0x08 NOBRIDGE - may be on or off
* 0x10 EPHEMERAL- MUST be off. Reply comments are not ephemeral.

## Server Used

These are posted to all of the author's Outbox servers.