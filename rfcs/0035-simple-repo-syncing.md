# RFC 35 - Easy hg/github → hg.mozilla.org syncing
* Comments: [#35](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/35)
* Proposed by: @bhearsum

# Summary

Support one-way syncing from existing hg or Github repos into Phabricator-enabled repositories.

## Motivation

A handful of projects (mostly notably PDF.js and NSS) regularly need to sync code into Gecko. In all cases, this is currently done by hand on laptops, making it annoying, error-prone, and low bus factor. Supporting this as a first class operation would make things better & easier for existing projects, and allow others to do so much more easily, enabling other use cases.

## Out of Scope

This RFC is intentionally tightly scoped to the currently known use cases. In particular, we are specifically _not_ supporting 2-way syncs or syncing things _into_ a Git or Github repository as part of this. These things may be considered in the future, but are explicitly out of scope for now.

# Details

Because this RFC is scoped to only support Phabricator-enabled repositories we can ignore some of the usual security concerns (because we won't have high-value credentials, just a Phabricator token). This also means we don't need to use something as complex as a scriptworker -- docker-worker should be perfectly fine for this use case. Most of the implemention work will be glueing existing things together. At a high level, we need a Task that:

* Takes the artifact from an upstream Task
* Unpacks it
* Clones the target repository
* Copies all or some of the contents to a target repository
* Runs `moz-phab` to create a Phabricator revision

Choosing to take an artifact from an upstream Task rather than simply copy files from a source repository clone is notable, as it provides the flexibility for projects to run build or other preprocessing steps before syncing is completed (which is a known use case for both PDF.js and NSS). If uses cases for more simple copies present themselves in the future, we may add support for the simpler operation as well - but it is not necessary at this time.

## Taskgraph

### New Docker Image

A new Docker image will be created that provides Mercurial (including any necessary tools to do fast clones), `moz-phab`, and a simple helper script that wraps the process described above. (This helper script primarily exists to ensure we never run into command line length limits.)

### Kind & Transforms

A new `kind` will be created that can create sync jobs. Sensible defaults will be used for descriptions, treeherder symbols, etc. It will run on docker-worker with the aforementioned image.

This kind will need a new transform that will take information provided by a project repository (see below), and use it to construct the right command to execute.

## Project Repositories

Will need to provide:

* Target repository URL
* Upstream task name and artifact name
* Source -> Target mapping, relatively to artifact root & target repository root, ie:

```yaml::
syncMappings:
    targetRepository: https://hg.mozilla.org/mozilla-central
    sourceArtifacts:
        - {buildTask}/buildArtifact.tar.bz2:
            src/: toolkit/components/pdfjs/content/build/
            web/: toolkit/components/pdfjs/content/web/
```

This information will be provided in the simplified Taskcluster config described by RFC XXX. (This needs updating after that RFC is opened.)

# Open Questions

* Does this enable single actor attacks? Ie: a privileged person in the PDF.js repo gets automation to create a Phab revision, and then reviews it themselves and lands it?
    * I think people can already land without review, so maybe it's effectively no different?

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

