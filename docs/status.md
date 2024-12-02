# Status and Development

### Page statuses

Page statuses move between these states

- TBD
- Incomplete
- Early Draft
- Draft (ready for others to review and comment on)
- Approved xN (N people have approved)
- Implemented xN (N implementations are known to exist)
- Superceded (some other page now supercedes this)

### Versions and Attribution

Many people are expected to contribute to Mosaic and as a result there are
various divergent viewpoints as to how Mosaic ought to be. As a result, we
label this edition as the Steve Farroll edition. You will see this at the
bottom of every page. Other Mosaic contributors can fork this and maintain
their own editions. The community will eventually settle on something
because people want to be compatible.

### Principles of Design

* The protocol must be *simple* enough for multiple implementations to
  be developed, but simplicity is not the only factor.
* The protocol must be *functional* enough to support a wide range of
  applications beyond just social media ones.
* It is okay to do things multiple ways so long as there is *one default*
  that all developers implement, and the rest of the "ways" are optional.
  For example, we can have multiple transports (WebSockets, WebTransport,
  even REST) without all the developers needing to move beyond WebSockets.
