# RFC 10 - Package caching proxy
* Comments: [#10](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/10)
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
* We should have a way to clear the cache (though, situations where the cache has to be manually cleared is a bug, and the cause of that situation should be addressed)
* Allows requesting a dependency by `(name, version, (optional) hash)`
    * Requiring hashing dependencies will be a follow-up RFC (`pip --require-hashes`, [`gradlew-witness`](https://github.com/signalapp/gradle-witness))

This package-caching proxy software should be tested against these requirements.
Once we're satisfied with its stability, we should gradually roll it out, one product at a time.
We will have two types of caching instances: one for untrusted builds (e.g.: staging, PRs) and another for trusted builds (e.g.: releases) 

We discussed using TC caches in our [earlier discussion](https://github.com/mozilla-releng/releng-rfcs/issues/4) and decided that it should be investigated in follow-up actions. It would be applied on top of the  caching proxy.

# Open Questions

* Who should set up and maintain this machine?
    * Proposal: RelOps creates instance, RelEng set up software and maintain
* I'm assuming [Cross-region caches](https://github.com/mozilla-releng/releng-rfcs/issues/4#issuecomment-452494805) means that there's a cache in each region. How should we handle propagating updates to the `$n` servers we'll have running? Leave to RelOps?
    * Will we want builds in region A use the caching proxy in region A? How do we want to set that up (since we can't hard-code package repositories in project build config in this case)?
* How should downtime be handled? 
    * Have two caching servers per region? Or:
    * Have projects depend on servers in multiple regions?

# Implementation



