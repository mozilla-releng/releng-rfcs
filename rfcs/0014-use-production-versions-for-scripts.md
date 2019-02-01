# RFC 14 - Use production semver versions for releng packages
* Comments: [#14](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/14)
* Proposed by: @mitchhentges, on behalf of discussion [in bug 1524248](https://bugzilla.mozilla.org/show_bug.cgi?id=1524248) 

# Summary

Bump all releng library and *script versions to >= `1.0.0`.

## Motivation

We're using our packages in production. According to [semver.org](https://semver.org/#how-do-i-know-when-to-release-100),
if software is being used in production, it should be >= `1.0.0`.

This will also allow us to convey intention in versions better (patches, deprecations, breaking changes, etc).

# Details

Bump all releng package versions to `1.0.0` (for those that aren't >= `1.0.0` already).
Follow [semver](https://semver.org/).

# Open Questions

# Implementation
