# Record Kinds

<status>PAGE STATUS: Early Draft</status>

Kinds are 8-byte unsigned integers in little-endian format.

This page is a Registry of kinds, their name, and the standard
that defines them.  Refer to their definitions in the standards that
define them.

|Kind|Name|Standard|Server Used|Dups|Read|Print|
|----|----|--------|-----------|-----------|---------|---------|
|270|[Key Schedule](keyschedule.md)|Core|Author Outbox|Replace|All|No|
|526|[Profile](profile.md)|Core |Author Outbox|Replace|All|No|
|796|[Microblog Root](microblog.md)|Social Media|Author Outbox|Unique|All|Yes|
|1052|[Reply Comment](reply_comment.md)|Social Media|Author Outbox<br>Parent Author Inbox|Unique|All|Yes|
|1308|[Blog Post](blog.md)|Social Media|Author Outbox|Unique|All|Yes|
|1564|[Chat Message](chat.md)|Social Media|Chat Server|Unique|All|Yes|
