# RFC 0041 - Mozilla Taskgraph

* Comments: [#40](https://github.com/mozilla-releng/releng-rfcs/issues/40)
* Proposed by: @ahal

## Summary

A new Python package called `mozilla-taskgraph` will be created to contain
Mozilla-specific Taskgraph logic currently residing in the [upstream Taskgraph
module][taskgraph] and various project Taskgraphs (especially in mobile).

### Motivation

This change will be beneficial both from the perspective of Mozilla as a
Taskgraph consumer, as well as Taskcluster as a standalone open-source project.

#### Motivation for Mozilla

Taskgraph is used across a variety of projects, each with unique needs and
unique tasks to support those needs. But there are also a lot of commonalities
between these projects. If Taskgraph logic should be shared across
projects, the only current place to do so is in the upstream [Taskgraph]
repository. Trying to add Mozilla-specific logic to this package, while at the
same time not breaking consumers outside of Mozilla is a big maintenance burden.

Having a dedicated place to add Mozilla logic will make it easier Releng and
devs using Taskgraph to organize code without worrying about impacting external
consumers.

#### Motivation for Taskcluster

Taskgraph is a unique tool and very good at generating a large graph of tasks.
In most ways it is the ideal complement for Decision tasks to call. The main
caveat is that it is currently only suitable to use at Mozilla, as there is a
lot of Mozilla-specific logic and terminology baked-in.

By pulling out the Mozilla-specific logic from core Taskgraph, Taskcluster can
finally point to Taskgraph as the recommended / standard way for consumers to
implement a Decision task.

## Details

### Project

The source for the new `mozilla-taskgraph` package will live under
`https://github.com/mozilla-releng/mozilla-taskgraph` and be published to Pypi
under the same name.

The `mozilla-taskgraph` project will use the same test, lint and code coverage
infrastructure as `taskcluster-taskgraph`.

### Dependency Management

The `mozilla-taskgraph` package will use the same mechanism as core Taskgraph
to manage its dependencies. As of this writing, this is via
`pip-compile-multi`.

The `mozilla-taskgraph` package will depend on the `taskcluster-taskgraph`
package and pin it using the [compatible release operator]. This will ensure
that consumers of `mozilla-taskgraph` can't accidentally use a version of
`taskcluster-taskgraph` that is incompatible with it.

The `mozilla-taskgraph` maintainers will need to create a new version whenever
a major upgrade of `taskcluster-taskgraph` is released.

### Source Layout

The `mozilla-taskgraph` package will have a layout similar to:

```bash
mozilla_taskgraph
├── morph.py
├── optimize
│   └── strategies.py
├── parameters.py
├── target_tasks.py
└── transforms
    ├── mobile
    │   ├── some_transform.py
    │   └── util
    └── scriptworker
    │   ├── some_transform.py
    │   └── util
    └── util

```

This is similar to core Taskgraph with a few differences:

1. Each "domain" will have a dedicated directory under the top-level
   `transforms` directory.
2. Utility files (if any) will live alongside the transforms that use them. If
   needed, a global utility module will exist for sharing across domains.

### Consumer Integration

The `mozilla-taskgraph` package will act as an optional extension that the
project repos can use. This means that the entrypoint for the Decision task
will remain core Taskgraph.

To accomplish this, `mozilla-taskgraph` will provide a `register()` function
that will set up required parameters, morphs or optimizations. It will be the
responsibility of a consuming project to call this function from within their
own `register()` function.

## Open Questions

### Will this add extra complexity?

* How much of a development burden will it add?
* Will it be harder for project owners to maintain?
* Could it impact a chemspill release?

### What is Mozilla-specific?

What is and isn't Mozilla-specific could be an open debate. The following list
is *very roughly* ordered from least to most controversial, and only contains
items that already exist in Taskgraph core:

* Anything related to scriptworkers
* Anything related to Treeherder
* Code review tasks
* The concept of `level`
* The concept of `trust_domains`
* The concept of `project`
* Actions
* Index route format (in `transforms/task.py`)
* Toolchains
* ...

## Implementation

* https://mozilla-hub.atlassian.net/browse/RELENG-1036

[taskgraph]: https://github.com/taskcluster/taskgraph
[compatible release operator]: https://peps.python.org/pep-0440/#compatible-release
