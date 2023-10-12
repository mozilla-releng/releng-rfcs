# RFC 33 - Release for Mozilla: Shared Trust Domain & Workers
* Comments: [#33](https://github.com/mozilla-releng/releng-rfcs/pull/33)
* Proposed by: @bhearsum

# Summary

Build and maintain a shared Trust Domain, Workers, and Scriptworkers on the Firefox CI cluster that any Mozilla project can use. Browser products will remain in their existing - separate - trust domain.

## Motivation

One of the barriers to entry for using Taskcluster is waiting on RelEng to create and deploy a new Trust Domain and Worker for a new project. Even when this takes less than a day to do (and it often takes longer), it's still something that needs to be waited on, and is slower than using CircleCI or Github Actions.

# Details

We will create a new trust domain and workers that are generally available for Mozilla employees and trusted volunteers to use. Specifically:

* A new Trust Domain (`mozilla`) that is not tied to a specific project or product
* New Workers for builds on Linux, macOS 10.15, and Windows Server 2012
    * A RelEng maintained Docker image will be provided for Linux
    * These will be created under a new `mozilla-1` provisioner
* New Workers for tests on Linux, macOS 10.15, and Windows 10
    * A RelEng maintained Docker image will be provided for Linux
    * These will be created under a new `mozilla-t` provisioner
* New Scriptworker instances for signing and mac-signing
	* These will be created under the existing `scriptworker-k8s` and `scriptworker-prov-v1` provisioners
	* Workers will be prefixed with `mozilla-t-`
    * mac-signing will run 10.14, like our other mac-signing workers (there's no known reason to upgrade)

Notably, we are only concerned with level 1 workers at this time, which means we can ignore things like scriptworkers that are only used when shipping. Level 3 workers will be dealt with at a later stage, and most likely will not use a shared trust domain or workers across projects.

Access to create and manage tasks on these new workers will be granted to anyone with `scm_level_1_github` or `scm_level_1`.

Going forward, we will ensure workers for other supported build or target platforms are added to this pool. (For example, when we add support for scheduling iOS tests in Taskcluster, that will be made available in the `mozilla-t` provisioner as well.)

## List of Pools

Pool ID                   | Purpose
====================================================================================
mozilla-1/linux           | Linux jobs
mozilla-1/linux-highcpu   | Linux jobs requiring more CPU resources
mozilla-1/win2012         | Windows Server 2012 jobs
mozilla-1/win2012-highcpu | Windows Server 2012 jobs requiring more CPU resources
mozilla-1/win10           | Windows 10 jobs
mozilla-1/macos-bigsur    | macOS 10.15 jobs
mozilla-t/signing         | Non-mac signing jobs
mozilla-t/mac-signing     | Mac signing jobs

## Hardware Machine Allocation

We will need hardware for 3 different pools, which will be allocated as noted below:
* 2 machines for macOS signing, running allocated from the existing production Firefox pool
* 3 machines for macOS builds, allocated from TBD
* 3 machines for macOS tests, allocated from TBD

When additional workers are needed in the future, they will be allocated from TBD.

## `v3` Taskcluster index format

The current `v2` index format only includes repository names as an identifier (not user or organization). This is generally not an issue for any of current trust domains (because they generally only support one project), but in this new pool where we have N projects, it introduces the potential for collisions or pollution between them. To ensure this isn't an issue we will introduce a new `v3` index format that includes the repository location in its path as well - including both domain and path. Examples include:
* `index.mozilla.v3.github.com.mozilla-mobile.mozilla-vpn-client.branch.main.latest.taskgraph.decision`
* `index.gecko.v3.hg.mozilla.org.releases.mozilla-beta.latest.firefox.decision`

This will require changes to a few things to support the new format:
* build-decision
* scriptworker
* taskgraph

Existing users of Taskcluster will not be required to upgrade to the v3 format.

# Open Questions

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>
