# RFC 0011 - Retire https://hg.mozilla.org/build
* Comments: [#11](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/11>)
* Proposed by: @tomprince

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
|braindump|Random releng scripts|Still in use|
|mozharness|Used for vcs syncing|Move to github|
|tools|Signing Server|Deprecated, Migrating to autograph|
|nagios-tools|Bouncer checkes|on buildbot-master01|
|relabs-puppet| |????|

Change the description of https://hg.mozilla.org/build to indicate that it of historical interest only.
The following repositories are obsolete and will remain where they are.

|Repository|Description|Status|
|----------|-----------|------|
|ash-mozharness| |Obsolete |
|autoland| |Obsolete|
|buildapi|Part of buildbot|Obsolete|
|buildbot|Part of buildbot|Obsolete|
|buildbot-configs|Part of buildbot|Obsolete|
|buildbotcustom|Part of buildbot|Obsolete|
|cloud-tools| |Moved to [githhub](https://github.com/mozilla-releng/build-cloud-tools)||
|compare-locales| |Moved to https://hg.mozilla.org/l10n/compare-locales/|
|mozpool|Part of buildbot|Obsolete|
|opsi-package-sources| |Obsolete|
|partner-repacks| |Moved to [github](https://github.com/mozilla-partners)|
|preproduction|Part of buildbot|Obsolete|
|puppet| |Moved to [github](https://github.com/mozilla-releng/build-puppet)|
|puppet-manifests| |Obsolete|
|rpm-sources| |Obsolete|
|serveS3| |Obsolete|
|slave_health|Part of buildbot|Obsolete|
|talos| |Moved [in-tree](https://hg.mozilla.org/mozilla-central/file/tip/testing/talos)|
|tupperware|Part of buildbot|Obsolete|
|twisted|Part of buildbot|Obsolete

# Open Questions

##### What to do with repositories still in use:
- braindump
- mozharness (vcs-sync)

##### Should we update all the remaining repositories with a note that they are obsolete?
(Many of repositories already have such a notice, sometimes with an outdated pointer to a newer repository)

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

