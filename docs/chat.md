# Chat

<status>PAGE STATUS: early draft</status>

Kind = 0x7 - chat message

## Payload

The payload follows the [Human Readable Content](human_readable_content.md) rules.

## Flags

* 0x01 ZSTD - may be on or off
* 0x02 FROMAUTHOR - may be on or off
* 0x04 TORECIPIENTS - may be on or off
* 0x08 NOBRIDGE - may be on or off
* 0x10 EPHEMERAL- SHOULD be on

## Tags

This MAY include up to one refer tag, either 0x6 or 0x7.

## Server Used

These are posted to a Chat Server.
