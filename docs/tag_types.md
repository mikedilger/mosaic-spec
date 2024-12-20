# Tag types

<status>PAGE STATUS: early draft</status>

Tag types are 2-byte (16-bit) unsigned integers in little-endian format.
Each application registers their tag types here to prevent collision.

Mosaic Core defines a few core tags in [Core Tags](core_tags.md).

|Tag Type|Name|Standard|
|--------|----|--------|
|0x1|Notify Public Key|Mosaic [Core Tags](core_tags.md)|
|0x2|Reply by Hash|Mosaic [Core Tags](core_tags.md)|
|0x3|Reply by Addr|Mosaic [Core Tags](core_tags.md)|
|0x4|Root by Hash|Mosaic [Core Tags](core_tags.md)|
|0x5|Root by Addr|Mosaic [Core Tags](core_tags.md)|
|0x8|Nostr Sister Event|Mosaic [Core Tags](core_tags.md)|
|0x10|Subkey|Mosaic [Core Tags](core_tags.md)|
|0x20|Content Segment: User Mention|Mosaic [Core Tags](core_tags.md)|
|0x21|Content Segment: Server Mention|Mosaic [Core Tags](core_tags.md)|
|0x22|Content Segment: Quote by Id|Mosaic [Core Tags](core_tags.md)|
|0x23|Content Segment: Quote by Addr|Mosaic [Core Tags](core_tags.md)|
|0x24|Content Segment: URL|Mosaic [Core Tags](core_tags.md)|
|0x25|Content Segment: Image|Mosaic [Core Tags](core_tags.md)|
|0x26|Content Segment: Video|Mosaic [Core Tags](core_tags.md)|

Application defined tags are TBD.
