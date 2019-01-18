# RFC 7 - Location of docs
* Comments: [#7](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/7)
* Proposed by: @mitchhentges

# Summary

* Update wiki.mozilla.org to describe a summary of the Releng team, links to more-detailed docs
* Remove [`moz-releng-docs.readthedocs.io`](https://moz-releng-docs.readthedocs.io/en/latest/) from [`mozilla/build-relengdocs`](https://github.com/mozilla/build-relengdocs)
* Create a new repository similar to [`taskcluster-docs`](https://github.com/taskcluster/taskcluster-docs). Move information from releng.readthedocs.io to here

## Motivation

We need a place to house docs that meets the following requirements:

* Smaller changes (e.g.: typo fixes) are possible without waiting for review
* Larger changes can be reviewed with a nice diff UI (GitHub's does this nicely)
* We can view history of docs
* The published docs are easily accessible (e.g.: $something.readthedocs.io)
* `moz-releng-docs.readthedocs.io` hasn't been updated for a few years, if we want to change the location of our docs, we wouldn't be harming our existing workflow too harshly

# Details

Follow the approach laid out in [`taskcluster-docs`](https://github.com/taskcluster/taskcluster-docs). Deploy these docs to a `readthedocs` site or equivalent.

# Open Questions

* Instead, should we continue using `mozilla/build-relengdocs`?
* Should we have releng-desktop.readthedocs.io and releng-mobile.readthedocs.io?
    * Should we have two separate docs sites, but tweak the above names?
    * Should we have all releng docs shared in a single site?
* How should we handle the [in-tree docs](https://firefox-source-docs.mozilla.org/taskcluster/taskcluster/release-promotion.html)
    * Move them to our new docs repository? Keep the in-tree docs, but have define how to decide whether new docs go in-tree or in-new-repository?
* Should we handle [`warrior-docs`](https://github.com/mozilla-releng/releasewarrior-2.0/tree/master/docs) as part of this RFC, or wait until Ship-it v2 to migrate into that?

# Implementation



