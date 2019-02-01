# RFC <number> - Use production versions for scripts
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @mitchhentges, on behalf of discussion [in bug 1524248](https://bugzilla.mozilla.org/show_bug.cgi?id=1524248) 

# Summary

Bump all script versions to >= `1.0.0`.

## Motivation

We're using our *scripts in production. According to [semver.org](https://semver.org/#how-do-i-know-when-to-release-100),
if software is being used in production, it should be >= `1.0.0`.

This will also allow us to convey intention in versions better (patches, deprecations, etc).

# Details

Bump all script versions to `1.0.0` (for those that aren't >= `1.0.0` already).
Follow [semver](https://semver.org/)

# Open Questions

# Implementation
