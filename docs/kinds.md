# Kinds Registry

This only covers Mosaic Core and Mosaic Social Media. Other applications should define
their internal kinds within their own specifications, and simply register an Application
ID with Mosaic.


|App |App Kind|Name              |Flags             |Server Used|
|----|--------|------------------|------------------|-----------|
|Core| 1 |[Key Schedule](keyschedule.md)|Replace, All, NoPrint|Author Outbox|
|Core| 2 |[Profile](profile.md)|Replace, All, NoPrint|Author Outbox|
|Social Media| 1 |[Microblog Root](microblog.md)|Unique, All, Print |Author Outbox|
|Social Media| 2 |[Reply Comment](reply_comment.md)|Unique, All, Print |Author Outbox<br>Parent Author Inbox|
|Social Media| 3 |[Blog Post](blog.md)|Unique, All, Print |Author Outbox|
|Social Media| 4 |[Chat Message](chat.md)|Unique, All, Print |Chat Server|
