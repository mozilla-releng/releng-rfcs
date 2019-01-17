# RFC <number> - Location of docs
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @mitchhentges

# Summary

* Update wiki.mozilla.org to describe a summary of the Releng team, links to more-detailed docs
* Remove releng.readthedocs.io from gecko in-tree
* Create a new repository similar to [`taskcluster-docs`](https://github.com/taskcluster/taskcluster-docs). Move information from releng.readthedocs.io to here


## Motivation

We need a place to house docs that meets the following requirements:

* Smaller changes (e.g.: typo fixes) are possible without waiting for review
* Larger changes can be reviewed with a nice diff UI (GitHub's does this nicely)
* We can view history of docs
* The published docs are easily accessible (e.g.: $something.readthedocs.io)

Most of releng's tooling is outside of the mercurial repo, so it's a bit of a hurdle to need to be trained on `hg` just to update docs, so moving these docs from the gecko tree would be beneficial.

# Details

Follow the approach laid out in [`taskcluster-docs`](https://github.com/taskcluster/taskcluster-docs). Deploy these docs to a `readthedocs` site or equivalent.

# Open Questions

* Should we have releng-desktop.readthedocs.io and releng-mobile.readthedocs.io?
    * Should we have two separate docs sites, but tweak the above names?
    * Should we have all releng docs shared in a single site?

# Implementation



