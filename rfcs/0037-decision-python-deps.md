# RFC 37 - Installing Decision task Python dependencies

# Summary

Outline a standard best practice for installing Python dependencies from a `.taskcluster.yml` file.

## Motivation

Many Taskcluster enabled repositories depend on Python via their use of Taskgraph. Taskgraph is
typically bootstrapped via the `run-task` script which clones the repository and checks out a
specified revision. This isn't ideal because it's not easy to tell how out of date the version of
Taskgraph being used is. It's also difficult to determine if there are any breaking changes while
updating. Both of these problems would be solved by using the [Pypi releases][0] which follow
semantic versioning.

Some repositories also depend on other libraries like `mozilla_version` or `redo`. These are
typically installed via `pip install` on the Decision task's command line. This isn't ideal because
we won't verify hashes and the version is often unpinned.

We should have a consistent method of installing Python dependencies which covers both use cases and
addresses the downsides of the current solutions for each.

# Details

The `run-task` script will be updated to support a `<REPO>_PIP_REQUIREMENTS` environment variable.
This will be a relative path from the root of the repository specified by `<REPO>` pointing to a
`requirements.txt` file. This `requirements.txt` file will be installed into the active Python
environment via `python -mpip install --require-hashes -r $<REPO>_PIP_REQUIREMENTS`.

Taskcluster consumers are free to use whichever tools they see fit to manage the `requirements.txt`
(as long as it has hashes).
E.g:

1. Generate it manually with help from `hashin`.
2. Compile it with `pip-tools`.
3. Export it from `poetry`.
4. Etc.

While this won't be enforced, the recommended location for this file will be
`taskcluster/requirements.txt`.

Requirements will be installed immediately prior to running the task's command.

## Multiple Repositories with Requirements

If `<REPO>_PIP_REQUIREMENTS` is specified for multiple repositories, all requirements files will be
installed at the same time and conflicting versions will result in an exception being raised. It is
up to the task's maintainers to ensure requirements files across repositories remain compatible with
one another.

## Testing Taskgraph Changes

Previously testing changes to Taskgraph within a consumer repo was easy. Just edit the repo to point
at `taskgraph-try` and select whichever revision is necessary. This proposal will complicate the
process a little bit.

The new recommended way to test changes is to use `pip`'s built-in [VCS support][1]. For example,
the following specifier could be used to test arbitrary Taskgraph revisions:

```
taskcluster-taskgraph@hg+https://hg.mozilla.org/ci/taskgraph@ae7697ec7905216c7245bafb8a9475355ea00a76
```

This can be compiled with hashes by `pip-compile` or used as a specifier to `hashin`. The new
process for testing against arbitrary Taskgraph revisions will be documented in Taskgraph's
documentation.

# Open Questions

1. If a repository does not wish to have a dependency on Pypi, we may need to provide a method to
   point at our internal Pypi instead. It's worth noting that any repo which installs Taskgraph via
   `pip install path/to/taskgraph` (which appears to be all of them) already depend on Pypi.

2. We'll need to be cautious not to break hooks, since previously the version of Taskgraph being
   used was baked into the `.taskcluster.yml`. Since changes to this file invalidate hooks, an
   update to Taskgraph will no longer force us to re-generate them. I don't believe this will cause
   any problems, but extra testing should be conducted here.


[0]: https://pypi.org/project/taskcluster-taskgraph/
[1]: https://pip.pypa.io/en/latest/topics/vcs-support/
