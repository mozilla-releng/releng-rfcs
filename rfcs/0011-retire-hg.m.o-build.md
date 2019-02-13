# RFC 0011 - Retire https://hg.mozilla.org/build
* Comments: [#11](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/11>)
* Proposed by: [@tomprince](https://github.com/tomprince)

# Summary

Mark https://hg.mozilla.org/build as obsolete, and move the still-active repositories elsewhere.
In particular, create a new top-level directory https://hg.mozilla.org/ci for firefox-ci related
repositories.

## Motivation

Most of the repositories in https://hg.mozilla.org/build are obsolete,
many being buildbot related, and most of the rest having moved elsewhere.
Additionally, the name "build" is more closely associated with the build team at Mozilla now.

The new taskcluster instance that ci-admin/ci-config are intended to manage has
been called the Firefox CI cluster (as has the part of the existing instance
that is used by Firefox). Those tools, as well as
extracted [taskgraph](https://bugzilla.mozilla.org/show_bug.cgi?id=1252144)
that is being worked on, should live somewhere that reflects that naming.


# Details

Create a new top-level directory on https://hg.mozilla.org/ci to house Firefox CI
related repositories, and move ci-admin and ci-config there.

The following repositories are still in use, but don't have a obvious home. How to handle them is currently an open question.

|Repository|Description|Status|
|----------|-----------|------|
|[braindump](https://hg.mozilla.org/build/braindump)|Random releng scripts|Still in use|
|[mozharness](https://hg.mozilla.org/build/mozharness)|Used for vcs syncing|Move to github|
|[tools](https://hg.mozilla.org/build/tools)|Signing Server|Deprecated, Migrating to autograph|
|[nagios-tools](https://hg.mozilla.org/build/nagios-tools)|Bouncer checkes|on buildbot-master01|
|[relabs-puppet](https://hg.mozilla.org/build/relabs-puppet)| |????|

Change the description of https://hg.mozilla.org/build to indicate that it of historical interest only.
The following repositories are obsolete and will remain where they are.

|Repository|Description|Status|
|----------|-----------|------|
|[ash-mozharness](https://hg.mozilla.org/build/ash-mozharness)| |Obsolete |
|[autoland](https://hg.mozilla.org/build/autoland)| |Obsolete|
|[buildapi](https://hg.mozilla.org/build/buildapi)|Part of buildbot|Obsolete|
|[buildbot](https://hg.mozilla.org/build/buildbot)|Part of buildbot|Obsolete|
|[buildbot-configs](https://hg.mozilla.org/build/buildbot-configs)|Part of buildbot|Obsolete|
|[buildbotcustom](https://hg.mozilla.org/build/buildbotcustom)|Part of buildbot|Obsolete|
|[cloud-tools](https://hg.mozilla.org/build/cloud-tools)| |Moved to [githhub](https://github.com/mozilla-releng/build-cloud-tools)||
|[compare-locales](https://hg.mozilla.org/build/compare-locales)| |Moved to https://hg.mozilla.org/l10n/compare-locales/|
|[mozpool](https://hg.mozilla.org/build/mozpool)|Part of buildbot|Obsolete|
|[opsi-package-sources](https://hg.mozilla.org/build/opsi-package-sources)| |Obsolete|
|[partner-repacks](https://hg.mozilla.org/build/partner-repacks)| |Moved to [github](https://github.com/mozilla-partners)|
|[preproduction](https://hg.mozilla.org/build/preproduction)|Part of buildbot|Obsolete|
|[puppet](https://hg.mozilla.org/build/puppet)| |Moved to [github](https://github.com/mozilla-releng/build-puppet)|
|[puppet-manifests](https://hg.mozilla.org/build/puppet-manifests)| |Obsolete|
|[rpm-sources](https://hg.mozilla.org/build/rpm-sources)| |Obsolete|
|[serveS3](https://hg.mozilla.org/build/serveS3)| |Obsolete|
|[slave_health](https://hg.mozilla.org/build/slave_health)|Part of buildbot|Obsolete|
|[talos](https://hg.mozilla.org/build/talos)| |Moved [in-tree](https://hg.mozilla.org/mozilla-central/file/tip/testing/talos)|
|[tupperware](https://hg.mozilla.org/build/tupperware)|Part of buildbot|Obsolete|
|[twisted](https://hg.mozilla.org/build/twisted)|Part of buildbot|Obsolete

# Open Questions

##### What to do with repositories still in use:
- braindump
- mozharness (vcs-sync)

##### Should we update all the remaining repositories with a note that they are obsolete?
(Many of repositories already have such a notice, sometimes with an outdated pointer to a newer repository)

# Implementation

* [Bug 1527721](https://bugzilla.mozilla.org/show_bug.cgi?id=1527721): Retire https://hg.mozilla.org/build
* [Bug 1525368](https://bugzilla.mozilla.org/show_bug.cgi?id=1525368): Please move `/build/ci-{ci-configuration,ci-admin}` to `/ci/ci-{ci-configuration,ci-admin}`.
