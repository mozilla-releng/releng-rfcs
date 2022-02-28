# RFC 5 - Shippable Builds
* Comments: [#5](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/5)
* Proposed by: @Callek

# Summary

Consolidate multiple build types and tests derived from those types into one solid answer for "is
this build/push suitable to ship to our users". And to maintain a build/test platform, where we can
get a fast answer for "Does this code landing need to be reverted".

## Motivation

We have a large array of build and test types, as well multiple variants of those types. This goal is to dedupe a
bunch of very-similar variations into a smaller set that answer the questions we care most about. Which should reduce costs
of running multiple things that serve similar purposes, and increase confidence in the builds that we do ship.

This work will, in addition, simplify the list of platform variants Sheriffs need to be aware of when triaging checkins, and
reduce complexity in the taskgraph, enabling us to iterate faster on other future improvements.

There will be a second stage to this work that entails what we call "Nightly Promotion" (RFC to come)

# Details

* We will create a new build variant `shippable` which will replace the variants of 'pgo' and 'nightly'.
* All tests currently running against either 'pgo' or 'nightly' build types will run against the new 'shippable' variant,
  and maintain current coverage.
* We will disable 'pgo' and 'nightly' builds and tests in favor of the new variant.
* Optimizations, such as SETA or SCHEDULES over when tests will run will remain unchanged.
* Shippable Builds are always intended to be able to ship to users, so we would have (where practical) zero configuration abnormalities
  compared to the builds we would want an end-user to have.

# Open Questions

* Conflicts with plan to reduce tests? [1]
  * I do not suspect there will be any negative interactions with this plan, and the reduction of tests will only further emphasize
    the benefits of shippable builds.

[1] - https://groups.google.com/d/msg/mozilla.dev.platform/0dYGajwXCBc/4cpYrUMyBgAJ (Joel's Plan)

# Implementation


* [bug 1452113 - implement shippable builds](https://bugzilla.mozilla.org/show_bug.cgi?id=1352113)
* [bug 1614970 - implement shippable builds, phase 2](https://bugzilla.mozilla.org/show_bug.cgi?id=1614970)

