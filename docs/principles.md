# Principles of Design

## Principles of Design

* The protocol must be *simple* enough for multiple implementations to
  be developed, but simplicity is not the only factor.
* The protocol must be *functional* enough to support a wide range of
  applications beyond just social media.
* The protocol should not impede high-performance high-throughput
  implementations.
* It is ok to do things multiple ways so long as there is *one default*
  that all developers implement, and the rest of the "ways" are truly
  OPTIONAL. For example, we can have multiple transports (QUIC, WebSockets)
  without all the developers needing to move beyond QUIC,
  so long as every implementation implements the QUIC transport.
* Code that isn't required by everybody should be defined outside of core
  as an extension, transport, or application, as these are all OPTIONAL.

[Rationale](rationale.md) used for various decisions is available in an
appendix.
