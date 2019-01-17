# RFC <number> - Package caching proxy
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Earlier discussion: [#4](https://github.com/mozilla-releng/releng-rfcs/issues/4)
* Proposed by: @mitchhentges

# Summary

Cache our dependencies for Maven/Gradle and Python in a "caching proxy" rather than putting them in Docker images.

## Motivation

This will provide the following advantages over our docker caching system:

* Updating product dependencies no longer requires re-building the docker image
    * Addresses the intermittent failures caused by package dependencies increasingly differing from the docker image each time it drifts out-of-date
* We can share a single cache between products, while only downloading necessary packages for each product

# Details

We will find off-the-shelf package-caching software (one for Maven/Gradle, one for Python) that meets the following requirements:

* If a dependency (that may or may not already be cached) is requested from multiple builds at the same time, the proxy should only request the dependency from the official repositories once
* Allows requesting a dependency by `(name, version, hash)` 
* (?) We should have a way to clear the cache (though, situations where the cache has to be manually cleared is a bug, and the cause of that situation should be addressed)

This package-caching proxy software should be tested against these requirements.
Once we're satisfied with its stability, we should gradually roll it out, one product at a time.

We discussed using TC indexes in our [earlier discussion](https://github.com/mozilla-releng/releng-rfcs/issues/4) and decided that it should be investigated in follow-up actions. It would be applied on top of the  caching proxy.

# Open Questions

* Is cache-clearing functionality necessary? Assuming working software, the only situation I can think of is if we pin a dependency version + hash, but the maintainer re-publishes on top of that version before our caching proxy downloads the pinned, replaced version. Perhaps this is a useful enough escape hatch to require in our caching software, hmm
* How do we want to handle checksums for dependencies?
    * In python, you can do `pip install --require-hashes ...` and it'll ensure that dependencies match your hashes
    * How should this be accomplished with `gradle`? Is it possible? Do we need to implement our own plugin?
        * I think checksum-pinning dependencies is valuable, and even without proxy caching, we're at risk (what if the official repositories are compromised right before the docker image is made?). Should setting this up block this RFC?

# Implementation



