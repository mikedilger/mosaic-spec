# Tag types

<status>PAGE STATUS: early draft</status>

Tag types are 2-byte (16-bit) unsigned integers in little-endian format.
Each application registers their indexable tag types here to prevent collision.

Applications should manage non-indexable tags within the payload.

Mosaic Core defines a few core tags in [Core Tags](core_tags.md).

|Tag Type|Name|Standard|
|--------|----|--------|
|0x0|PADDING MARKER|This is not a tag, it signifies a padding area|
|0x1|Notify Public Key|Mosaic [Core Tags](core_tags.md)|
|0x2|Reply|Mosaic [Core Tags](core_tags.md)|
|0x3|Root|Mosaic [Core Tags](core_tags.md)|
|0x8|Nostr Sister Event|Mosaic [Core Tags](core_tags.md)|
|0x10|Subkey|Mosaic [Core Tags](core_tags.md)|
|0x20|Content Segment: User Mention|Mosaic [Core Tags](core_tags.md)|
|0x21|Content Segment: Server Mention|Mosaic [Core Tags](core_tags.md)|
|0x22|Content Segment: Quote|Mosaic [Core Tags](core_tags.md)|
|0x24|Content Segment: URL|Mosaic [Core Tags](core_tags.md)|
|0x25|Content Segment: Image|Mosaic [Core Tags](core_tags.md)|
|0x26|Content Segment: Video|Mosaic [Core Tags](core_tags.md)|

Application defined tags are TBD.
