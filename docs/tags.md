# Tags

<status>PAGE STATUS: early draft</status>

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
* `[2:4]` - The type of the tag
* `[4:]` - The value, which is at most 65,532 bytes long.

Tag values are generally considered opaque at this layer.

Unlike nostr, tags do not have multiple value fields. Applications can encode data
into the 65,532 bytes however they wish. But applications SHOULD expect tags to
be indexed by their type and some prefix of their value (of unspecified length but
it SHOULD include at least 32 bytes), so uniqueness SHOULD frontloaded.
