# Tags

<t>Tags</t> [<sup>rat</sup>](rationale.md#tags) contain data that is indexed and searchable.
All tags are searchable on servers. If an application requires unsearchable tags,
these can be defined within that application's payload.

Tags are laid out as follows:

```text
0       2        4         65536 max
+-------+--------+----------+
| len   | type   | value ...|
+-------+--------+----------+
```

* `[0:2]` - The length of the tag value as a little-endian unsigned integer.
             This length includes the len and type bytes and thus MUST be
             at least 4.
* `[2:4]` - The type of the tag. All tag types are listed in the [Tag types](tag_types.md) registry.
* `[4:]` - The value, which is at most 65,532 bytes long.

Tag values are generally considered opaque at this layer, except for [Core tags](core_tags.md) with the Core Application.

Unlike nostr, tags do not have multiple value fields. Applications can encode multiple
fields into the 65,532 data bytes however they wish.

Applications SHOULD expect tags to be indexed by their type and some prefix of their value
(of unspecified length, but it SHOULD include at least 32 bytes). As a consequence, tags should
be defined to front-load their unique aspects within the first 32 bytes of their value to
maximize the effectiveness of indexes.
