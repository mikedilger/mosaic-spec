# Status and Development

## Page statuses

Page statuses move between these states

- TBD
- Incomplete
- Early Draft
- Draft (ready for others to review and comment on)
- Approved xN (N people have approved)
- Implemented in core
- Implemented in core + xN (N implementations are known to exist)
- Superceded (some other page now supercedes this)

Note that developers are encouraged to comment on and discuss pages in any
status, they don't have to wait for draft status.

## Versions and Attribution

Many people are expected to contribute to Mosaic and as a result there are
various divergent viewpoints as to how Mosaic ought to be. As a result, we
label this edition as the Steve Farroll edition. You will see this at the
bottom of every page. Other Mosaic contributors can fork this and maintain
their own editions. The community will eventually settle on something
because people want to be compatible.

## Principles of Design

* The protocol must be *simple* enough for multiple implementations to
  be developed, but simplicity is not the only factor.
* The protocol must be *functional* enough to support a wide range of
  applications beyond just social media.
* The protocol should not impede high-performance high-throughput
  implementations.
* It is ok to do things multiple ways so long as there is *one default*
  that all developers implement, and the rest of the "ways" are optional.
  For example, we can have multiple transports (WebSockets, WebTransport)
  without all the developers needing to move beyond WebSockets,
  so long as every implementation implements the WebSockets transport.
* Code that isn't required by everybody should be defined outside of core
  as an extension, transport, or application, as these are all optional.


## Core Library

This specification is being developed in parallel to a
[core library](https://github.com/SteveFarroll/mosaic-core). The
findings from development feed back into this specification.
