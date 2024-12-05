# Record Kinds

<status>PAGE STATUS: Early Draft</status>

Kinds are 4-byte unsigned integers in little-endian format.

This page is a Registry of kinds, their name, and the standard
that defines them.  Refer to their definitions in the standards that
define them.

|Kind|Name|Standard|Server Used|
|----|----|--------|-----------|
|0x1|Key Schedule|Mosaic Core [Key Schedule](keyschedule.md)|Outbox|
|0x2|Profile|Mosaic Core [Profile](profile.md)|Outbox|
|0x3|Microblog Root|Mosaic Social Media [Microblog Root](microblog.md)|Outbox|
|0x4|Reply Comment|Mosaic Social Media [Reply Comment](reply_comment.md)|Outbox & Parent Author Inbox|
|0x5|Blog Post|Mosaic Social Media [Blog Post](blog.md)|Outbox|
|0x6|Chat Message|Mosaic Social Media [Chat Message](chat.md)|Chat Server|
