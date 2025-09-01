# Profile (kind)

A profile record contains user profile information.

A profile record MUST be considered invalid if the signing key and the
master key are not identical.

## Kind

**Applicaton**: 0 (Mosaic Core)

**App Kind**: 2 (Profile)

**Flags**: 0x000e = *Replaceable*, *Everybody*, *NotPrintable*

Representation = 0x0000_0000_0002_000e

## Tags

No specific tags are defined.

## Payload

Payload is a [CBOR](https://www.rfc-editor.org/rfc/rfc8949.html) Map.
All field names (map keys) are snake-case Text.

The field `name` is REQUIRED. All other fields are OPTIONAL.

The following fields are available:

* `name` - (Text, REQUIRED) this is the user's name, used for `@name` tagging,
  containing only typeable ASCII characters. These cannot be guaranteed
  to be unique. Users are encouraged to choose names that are uncommon
  if not unique. Clients are encouraged to provide a way for users to
  disambiguate name collisions such as via a
  [Petname](https://en.wikipedia.org/wiki/Petname) system,
  which is out of scope of this specification.
* `display_name` - (Text) this is a name for display purposes which can contain
  any sort of UTF-8 character. Clients are encouraged to also display
  `name` so that users can tag them with `@name` syntax.
* `about` - (Text) This is prose where the user can describe themself.
* `avatar` - (Bytes) This is an avatar image in the WebP image format.
  Animated WebP is allowed.
  It MUST be no larger than 320 pixels in either dimension.
  It SHOULD be dimensionally square.
  Note that clients MAY crop the image into a circle when displaying it.
  [<sup>rat</sup>](rationale.md#avatar).
* `website` - (Text) This is a URL to the user's website.
* `banner` - (Bytes) This is an image in WebP image format.
  It MUST be no larger than 1280 wide, and it MUST be no larger than 720 high.
* `org` - (Boolean) A boolean designating that this profile represents an organisation
* `bot` - (Boolean) A boolean designating that this profile is a bot
* `lud16` - (Text) This is a lightning payment address
  defined by [LUD-16](https://github.com/lnurl/luds/blob/luds/16.md)

Additional fields MAY be used.

Note that the The maximum record size is one megabyte, and that the profile data
must fit within this one megabyte, including the binary images.

## Flags

There are no special flag restrictions.

## Server Used

These are posted to all of the author's Outbox servers.
