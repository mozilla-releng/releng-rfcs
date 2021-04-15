# RFC 33 - Release for Mozilla: Shared Trust Domain & Workers
* Comments: [#33](https://github.com/mozilla-releng/releng-rfcs/pull/33)
* Proposed by: @bhearsum

# Summary

Build and maintain a shared Trust Domain, Workers, and Scriptworkers on the Firefox CI cluster that any Mozilla project can use.

## Motivation

One of the barriers to entry for using Taskcluster is waiting on RelEng to create and deploy a new Trust Domain and Worker for a new project. Even when this takes less than a day to do (and it often takes longer), it's still something that needs to be waited on, and is slower than using CircleCI or Github Actions.

# Details

We will create a new trust domain and workers that are generally available for Mozilla employees and trusted volunteers to use. Specifically:

* A new Trust Domain (`mozilla`) that is not tied to a specific project or product
* New Workers for builds on Linux, macOS 11.0, and Windows Server 2012
    * These will be created under a new `mozilla-1` provisioner
* New Workers for tests on Linux (through developer provided Docker images), macOS 11.0, and Windows 10
    * These will be created under a new `mozilla-t` provisioner
* New Scriptworker instances forsigning and mac-signing
	* These will be created under the existing `scriptworker-k8s` and `scriptworker-prov-v1` provisioners
	* Workers will be prefixed with `mozilla-1-`

Notably, we are only concerned with level 1 workers at this time, which means we can ignore things like scriptworkers that are only used when shipping. Level 3 workers will be dealt with at a later stage.

Access to create and manage tasks on these new workers will be granted to anyone with `scm_level_1`.

Going forward, we will ensure workers for other supported build or target platforms are added to this pool. (For example, when we add support for scheduling iOS tests in Taskcluster, that will be made available in the `mozilla-t` provisioner as well.)

# Open Questions

* Are we happy with the new trust domain name & provisioners for the workers?
* Where did we get the macOS hardware for the build, test, and signing pools?
	* New or pull from existing pools?
	* How many machines do we need in each hardware pool?
* Is macOS 11.0 the right version to use for build and test?
* Are there other test platforms or scriptworkers we should support?
* Is `scm_level_1` the right group to use, or do we need a new one for this purpose?

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

