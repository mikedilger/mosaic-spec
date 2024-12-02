# Blog

<status>PAGE STATUS: early draft</status>

App ID = 0x6 - Blog post

## Flags

* 0x01 ZSTD - may be on or off
* 0x02 FROMAUTHOR - may be on or off to control distribution.
* 0x04 TORECIPIENTS - MUST be off. Blogs are public.
* 0x08 NOBRIDGE - may be on or off
* 0x10 EPHEMERAL- MUST be off. Blogs are not ephemeral.

## Tags

Blogs SHOULD not have the following tags: 0x1, 0x2, 0x3, 0x4, 0x5, 0x6.

Blogs MAY have 0x7.
