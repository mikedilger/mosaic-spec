# Profile (kind)

<status>PAGE STATUS: early draft</status>

A profile record contains user profile information.

A profile record MUST be considered invalid if the signing key and the
master key are not identical.

## Kind

**Applicaton**: Mosaic Core

**App Kind**: 2 (Profile)

**Flags**: *Replaceable*, *Everybody*, *NotPrintable*

Representation = `[0, 0, 0, 0, 0, 0, 2, 14]`

## Tags

No specific tags are defined.

## Payload

Payload is a [CBOR](https://www.rfc-editor.org/rfc/rfc8949.html) map.
All field names are snake-case.

The only required field is `name`.

The following fields are available:

* `name` - (REQUIRED) this is the user's name, used for `@name` tagging,
  containing only typeable ASCII characters. These cannot be guaranteed
  to be unique. Users are encouraged to choose names that are uncommon
  if not unique. Clients are encouraged to provide a way for users to
  disambiguate name collisions such as via a
  [Petname](https://en.wikipedia.org/wiki/Petname) system,
  which is out of scope of this specification.
* `display_name` - this is a name for display purposes which can contain
  any sort of UTF-8 character. Clients are encouraged to also display
  `name` so that users can tag them with `@name` syntax.
* `about` - This is prose where the user can describe themself.
* `avatar` - This is a URL to an avatar (e.g. a profile picture) in a
  standard format (PNG, JPEG, SVG, WebP, GIF, BMP or ICO) expected to be
  dimensionally square and no bigger than 512x512.
* `website` - This is a URL to the user's website.
* `banner` - This is a URL to a background picture in a standard format
  (PNG, JPEG, SVG, WebP, GIF, BMP or ICO) expected to be of size 1024x768
* `org` - A boolean designating that this profile represents an organisation
* `bot` - A boolean designating that this profile is a bot
* `lud16` - This is a lightning payment address
  defined by [LUD-16](https://github.com/lnurl/luds/blob/luds/16.md)

Additional fields MAY be used.

## Flags

There are no special flag restrictions.

## Server Used

These are posted to all of the author's Outbox servers.
