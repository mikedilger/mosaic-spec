# Notation

<status>PAGE STATUS: early draft</status>

## Rationale links

When you see a <t>superscript</t> [<sup>rat</sup>](rationale.md), the "rat" is a link
to the rationale behind this. Rationale is kept so we can remember why decisions were
made, but kept separate to not clutter the specification.

### MUST, SHOULD, and MAY

The capitalized words "MUST", "REQUIRED", "SHALL", "MUST NOT" and
"SHALL NOT" indicate an absolute requirement in order to be in compliance with this
specification, and violation means the software does not comply with the Mosaic
specification.

The capitalized words "SHOULD", "SHOULD NOT", and "RECOMMENDED" indicate
strong advice for the default case, but there may be valid exceptions and software
that does otherwise is not in violation of the specification.

The capitalized words "MAY" and "OPTIONAL" indicate a choice that is truly
optional.

These definitions are differently worded, but are not meant to be functionally different
from [rfc2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Byte Slice Notation

Byte slice notation `[m:n]` indicates the bytes including `m` up to and
including the byte `n-1` but not including the byte `n`. For example `[8:12]`
represents bytes 8, 9, 10 and 11.

Byte slices that are missing a beginning such as `[:64]` start at 0.

Byte slices that are missing an ending such as `[112:]` continue until the
end of the data.
