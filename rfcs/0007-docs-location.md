# RFC 7 - Location of docs
* Comments: [#7](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/7)
* Proposed by: @mitchhentges

# Summary

For all of our releng documentation, ensure that it's possible to find via `mozilla/build-relengdocs`.
This may be accomplished for each piece of documentation by either:
* Storing it in `mozilla/build-relengdocs`
* Storing it elsewhere, but linking to the published location from `mozilla/build-relengdocs`

## Motivation

This RFC aims to ease the task of answering "where are the existing docs for `$product` or `$workflow`" by ensuring that
docs are findable from a central location (`mozilla/build-relengdocs`).

Additionally, there's some minor fragmentation and documentation accessibility that can be improved.

# Details

* Adds links to `mozilla/build-relengdocs` for documentation that is outside of that repository (for example, geckoview)
* Move releaseduty docs ([in here](https://github.com/mozilla-releng/releasewarrior-2.0/tree/master/docs)) to `mozilla/build-relengdocs`
* Update https://wiki.mozilla.org/Main_Page - move technical information to `mozilla/build-relengdocs`
    * Note: leave releasewarrior documentation ([also in here]((https://github.com/mozilla-releng/releasewarrior-2.0/tree/master/docs))),
since it's deprecated. Once the last consumer (Thunderbird) moves to ship-it v2, these docs can be removed.
* Add ability to write docs in `mozilla/build-relengdocs` in markdown (while still allowing `.rst`)
    * Motivation: these docs are on Github and using the Github workflow, and markdown is the common markup language on the platform.
    additionally, [`scriptworker` docs](https://github.com/mozilla-releng/scriptworker/tree/master/docs) are already a combination of both
    `.rst` and `.md`

When deciding where to add new documentation, the following information can be helpful:
* The documentation target project's location - if the project is in-tree, perhaps the docs should be as well
* Do docs for a related project/workflow already exist? If so, perhaps the new docs should be close by

# Open Questions

* Should we move all out-of-tree docs to be under a single domain? We could have our 
top-level `releng.mozilla.org`, plus our projects underneath it: `scriptworker.releng.mozilla.org`, 
`mozapkpublisher.releng.mozilla.org`, etc

# Implementation



