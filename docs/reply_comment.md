# Reply Comment (kind)

## Kind

**Applicaton**: 1 (Mosaic Social Media)

**App Kind**: 2 (Reply Comment)

**Flags**: 0x001c = *Unique*, *Everybody*, *Printable*

Representation = 0x0000_0001_0002_001c

## Tags

This MUST include exactly one reply tag, either 0x2 or 0x3.

This MUST include exactly one root tag, either 0x2 or 0x3.

## Payload

The payload follows the [Human Readable Content](human_readable_content.md) rules.

## Flags

There are no special flag restrictions.

## Server Used

These are posted to all of the author's Outbox servers and also to all
of the parent Record author's Inbox servers.
