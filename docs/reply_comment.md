# Reply Comment (kind)

## Kind

**Applicaton**: Mosaic Social Media

**App Kind**: 2 (Reply Comment)

**Flags**: *Unique*, *Everybody*, *Printable*

Representation = `[0, 0, 0, 0, 1, 0, 2, 28]`

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
