# RFC 34 - Automatically update ci-configuration for new repositories
* Comments: [#34](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/34)
* Proposed by: @bhearsum

# Summary

Automatically configure the Firefox CI cluster for new repositories when they install the Firefox CI Taskcluster integration.

## Motivation

Currently, ci-configuration changes must be made manually for every new repository that wants to use the Firefox CI Taskcluster instance. These changes are not typically difficult to make, but they can only be done by a small set of authorized people. This is a barrier to entry for people using Taskcluster (especially when you compare to bootstrapping something like CircleCI or Github Actions).

# Details

Implementing this will involve small to medium sized changes to the ci-configuration repository, the ci-admin tool, taskcluster-github, and hg.mozilla.org hooks. It will also require a small new tool to make the actual changes.

## Overview
When a user installs the Firefox CI Taskcluster integration, Taskcluster-Github will receive either a `installation` or `installation_repository` event. In response it will create a Pulse message ontaining repository, user or organization, and sender information. A new tool, `auto-ci-config`, will receive these messages and update ci-configuration in response to them (more on that below). Finally, CloudOps' Jenkins instance will apply the ci-configuration change, as it does for any other ci-configuration change.

## auto-ci-config

This tool will listen for events on a Pulse queue that receives messages whenever a new Firefox-CI Taskcluster installation happens. In response, it will:
* Finds the sender in the Mozilla Person API (via queries like https://people.mozilla.org/api/v4/search/simple/?q=sciurus&w=staff)
* Verifies that they are in the `taskcluster_users` LDAP group. If not, exit
* Finds the default branch of the repository by querying the Github API
* Creates and pushes the necessary ci-configuration change

## ci-configuration
A new file (`projects-automanaged.yml`) will be added that is used for automanaged Github projects. It will look similar to projects.yml except that it will contain a top-level key of `github-projects`. Additionally, a key called `repositories` will be required, which will contain a list of github repositories that should be configured with the noted parameters. In essence, this is an inverted and limited form of what is already used in `projects.yml`. This structure implies that all projects listed in it will share the same configuration. Projects that need separate configuration may graduate to `projects.yml` later.

Proposed format:
```
github-projects:
    repositories:
        https://github.com/mozilla/mozilla-vpn-client:
            default_branch: main
    features:
        taskgraph-actions: true
        github-taskgraph: true
        trust-domain-scopes: true
```

We will need a couple of small changes to grants to:
* Allow decision tasks in the automanaged projects to have the scopes they need to run tasks for automanaged projects
* Allow anyone in the `taskcluster_users` LDAP group to create, rerun, etc. tasks for automanaged projects

Notably, this means that users from project A can cancel, rerun, etc. tasks for project B. This is considered acceptable for level 1 access.

## ci-admin

ci-admin will need support adding for processing the new `projects-automanaged.yml` file. This will include hardcoding `level` to `1` and `trust_domain` to `mozilla` to ensure that all automanaged projects use these values in their scopes and grants (which are the only valid values).

## hg.mozilla.org

We will need a hook that prevents the account that will be pushing these changes from modifying files other than `projects-automanaged.yml`, to prevent any compromise of that system from making any changes to Firefox, Fenix, or other more high risk parts of ci-configuration. There is existing prior art on this sort of limitation, so this is mostly configuration, not development.

## taskcluster-github

Taskcluster-Github will be updated to support received `installation` and `installation_repository` events, and creating Pulse messages in response to them.

# Open Questions

* Are we OK with something pushing ci-config changes fully automatically, with it restricted to one particular file?
    * Hal was OK with the general idea of this.
* Is an LDAP group the right thing to use? Or should we use a Mozillians group? Some other way of authorizing the request?
* Which `features` will we enable for automanaged projects?
* Should `features` be hardcode in `ci-admin` like `level` and `trust_domain` are?

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>
