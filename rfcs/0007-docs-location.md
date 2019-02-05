# RFC 7 - Location of docs
* Comments: [#7](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/7)
* Proposed by: @mitchhentges

# Summary

When adding new documentation:
* If it's for an in-tree project:
    1. Add to [in-tree docs](https://firefox-source-docs.mozilla.org/taskcluster/taskcluster/release-promotion.html)
    2. Add a link to the in-tree docs in [`mozilla/build-relengdocs`](https://github.com/mozilla/build-relengdocs) 
* Otherwise:
    1. Add to [`mozilla/build-relengdocs`](https://github.com/mozilla/build-relengdocs)
    
When looking for documentation for an existing project:
1. Go to [`mozilla/build-relengdocs`](https://github.com/mozilla/build-relengdocs)
2. If it links to in-tree docs, follow the link

As part of this RFC, we'll coalesce existing documentation into the in-tree and `mozilla/build-relengdocs` docs so
they match the decision tree above.

Also, update wiki.mozilla.org to describe a summary of the Releng team and link to `mozilla/build-relengdocs`

## Motivation

This RFC aims to directly answer the question "where should I put my docs for `$productA`" or "where are the existing docs 
for `$productB`".

Additionally, this RFC tries to balance the benefit of having centralized documentation with the ability to have docs
on the same platform as the project they're documenting.

Requirements for documentation platforms:
* Smaller changes (e.g.: typo fixes) are possible without waiting for review
* Larger changes can be reviewed with a nice diff UI (GitHub's does this nicely)
* We can view history of docs
* The published docs are easily accessible (e.g.: $something.readthedocs.io)
* Process of updating docs shouldn't be too heavy: making it tough to edit docs may increase the chances that docs will become out-of-date

# Details

Have two platforms for documentation: [in-tree docs](https://firefox-source-docs.mozilla.org/taskcluster/taskcluster/release-promotion.html)
and [`mozilla/build-relengdocs`](https://github.com/mozilla/build-relengdocs).

* Add links from `mozilla/build-relengdocs` for each project documented in in-tree docs
* Comb through `mozilla/build-relengdocs` and remove obsolete documentation
* Add ability to write docs in `mozilla/build-relengdocs` in markdown
    * Motivation: these docs are on Github and using the Github workflow, and markdown is the common markup language on the platform
* Move releaseduty docs ([in here](https://github.com/mozilla-releng/releasewarrior-2.0/tree/master/docs)) to `mozilla/build-relengdocs`
* Leave releasewarrior documentation ([also in here]((https://github.com/mozilla-releng/releasewarrior-2.0/tree/master/docs))),
since it's deprecated. Once the last consumer (Thunderbird) moves to ship-it v2, the releasewarrior docs can be removed.

# Open Questions

* Are we happy with being able to use markdown for `build-relengdocs`?
    * Should we convert existing `.rst` docs in `build-relengdocs` to markdown?

# Implementation



