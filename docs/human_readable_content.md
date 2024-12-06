# Human Readable Content

Records kinds which contain payloads of human readable content work as follows.

The payload MUST be a valid UTF-8 encoded string.

Any part of the intended content that is meant to have machine-readable meaning must
not be in the payload, but instead specified in a content segment tag, which indicates
the kind of content, the character (not byte) offset into the payload where this content
should be rendered, and the value of the content.  Multiple kinds of content segment
tags are defined including:

* [Content Segment: User Mention](core_tags.md#content-segment-user-mention) for things like `@user`.
* [Content Segment: Server Mention](core_tags.md#content-segment-server-mention) for mentioning a server.
* [Content Segment: Quote by id](core_tags.md#content-segment-quote-by-id) for quoting other records.
* [Content Segment: Quote by addr](core_tags.md#content-segment-quote-by-addr) for quoting other records.
* [Content Segment: URL](core_tags.md#content-segment-url) for inserting a URL to a website.
* [Content Segment: Image](core_tags.md#content-segment-image) for inserting an image.
* [Content Segment: Video](core_tags.md#content-segment-video) for inserting a video.
