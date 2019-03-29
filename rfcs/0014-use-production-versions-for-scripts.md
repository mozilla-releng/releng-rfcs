# RFC 14 - Use production semver versions for production releng packages
* Comments: [#14](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/14)
* Proposed by: @mitchhentges, on behalf of discussion [in bug 1524248](https://bugzilla.mozilla.org/show_bug.cgi?id=1524248) 

# Summary

Bump all production releng library and *script versions to >= `1.0.0`.

## Motivation

We're using some of our packages in production. According to [semver.org](https://semver.org/#how-do-i-know-when-to-release-100),
if software is being used in production, it should be >= `1.0.0`.

This will also allow us to convey intention in versions better (patches, deprecations, breaking changes, etc).

# Details

For all of our releng packages that we use in production, bump their versions to `1.0.0` (for those that aren't >= `1.0.0` already).
Follow [semver](https://semver.org/).
Importantly, this means that a backwards-incompatible change to _any_ public-facing functionality (e.g.: *script config 
or task payload structure) must be followed by a major version bump.

# Open Questions

# Implementation

* [Tracking bug](https://github.com/mozilla-releng/releng-rfcs/issues/19)
