# Status and Development

## Versions and Attribution

Many people are expected to contribute to Mosaic and as a result there are
various divergent viewpoints as to how Mosaic ought to be. As a result, we
label this edition as the Mike Dilger edition. You will see this at the
bottom of every page. Other Mosaic contributors can fork this and maintain
their own editions. The community will eventually settle on something
because people want to be compatible.

Version numbers have MAJOR.MINOR.PATCH format which carry the following meaning:

- **MAJOR**: This is a breaking change. Software should support all major versions.
- **MINOR**: Functionality has changed. But this is not a breaking change.
- **PATCH**: Something has changed, but it is not a functional change.

Major Version 0 is pre-release. There will be a long series of breaking changes
in version 0 without updating the major version from 0.  Any deployments should
be for testing purposes, be considered provisional and expect breaking changes.

Version 1.x will be the first stabilized and frozen release. All future
versions are intended to support it for backwards compatibility.
