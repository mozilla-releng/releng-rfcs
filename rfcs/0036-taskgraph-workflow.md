# RFC 36 - Enhance Taskgraph to support the common build → test workflow
* Comments: [#36](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/36)
* Proposed by: @bhearsum

# Summary

Enchance Taskgraph to support the common build → test workflow without requiring custom code in a project repository.

## Motivation

One of the barriers to entry for using Taskcluster is getting usable configs and support files ready and working properly. This is needed even when a project just needs simple builds and tests, and can be both intimidating and complex. Adding support for this common workflow in Taskgraph will make it much quicker and easier to onboard with projects.

# Details

## Division of Responsibilities

A conceit of this RFC is that while no custom _code_ should be needed to do simple builds and tests, we will still require some minimal amount of _config_. Effectively this will shift most of the responsibility and complexity for simple builds and tests from project repos to Taskgraph itself.

Project repositories will only need to provide simple information that should be mostly obvious to their owners:
* A list of build & test platforms, as well as the following for each:
    * Command used to build or test
    * Worker type and docker image name (when applicable)
    * Artifacts to upload
    * Dependent builds or tests, and artifacts required from them

## New Taskcluster Config

A new, simplified, Taskcluster config will be created. It will be located in `.taskcluster/config.yml` and contain:
* A reference to another config file, which will contain all of the things normally found in a `.taskcluster.yml`
* A list of tasks using a new, simplified schema

Here's an example:

```yaml::
version: 1
baseConfig: https://some/releng/owned/thing/taskcluster.yml-v1
extraTransforms:
    - treeherder
tasks:
    build-linux64:
        workerType: b-linux
        docker-image: build
        command: make build
        produces:
            linux64.tar.bz2: build/linux64.tar.bz2
            tests.tar.bz2: build/tests.tar.bz2
        notify:
            slack:
                - CKM7DLL67 # #myproject
            matrix:
                - !jmMnZXHjnLBJCfUIWh:mozilla.org # #myproject
            
    test-linux64:
        workerType: t-linux
        docker-image: ubuntu1804
        consumes:
            build-linux64:
                - linux64.tar.bz2
                - tests.tar.bz2
        command: tar -jvxf tests.tar.bz2 && run_tests.py --build linux64.tar.bz2
```

This config is explicitly trying to be as simple as possible by doing a few things:
* Assuming that tasks are always a command running on a given worker/image
* Not structurally separating task metadata and payload data
* Explicitly _not_ supporting JSON-e. Complex use cases should be moved to more traditional ways of defining tasks (see below).

In some ways this is an even more simplified version of the `run-task` job type used in the Gecko tree.

The full schema for version 1 is as follows:
```yaml::
required:
    - version
    - baseConfig
    - tasks

properties:
    version:
        type: integer
        description: Config Schema Version
        default: 1
        enum:
            - 1

    baseConfig:
        type: string
        description: A link to the .taskcluster.yml to use when generating tasks for this repository.
        default: https://some/releng/owned/thing/taskcluster.yml-v1

    extraTransforms:
        type: array
        description: A list of extra transforms to apply to the Tasks in this config
        items:
            type: string
            description: Transform name
            examples:
                - treeherder

    tasks:
        type: object
        description: A set of tasks tasks to run in response to pushes and pull requests, keyed by task name
        patternProperties:
            ^.*$:
                type: object
                required:
                    - command
                    - workerType
                properties:
                    produces:
                        type: object
                        description: Artifacts produced by the task, keyed by what they should be called
                        patternProperties:
                            ^.*$:
                                type: string
                                description: Path to the artifact, relative to the root of the build directory

                    command:
                        type: string
                        description: Command used to run the Task. Complex commands should generally be avoided, and moved into a checked-in script.

                    env:
                        type: object
                        description: Environment variables to set for the Task, keyed by variable name
                        patternProperties:
                            ^.*$:
                                type: string
                                description: Variable value

                    notify:
                        type: object
                        properties:
                            matrix:
                                type: array
                                items:
                                    type: string
                                    description: Matrix channel ID to send a notification to on task failures

                            slack:
                                type: array
                                items:
                                    type: string
                                    description: Slack channel ID to send a notification to on task failures

                    dockerImage:
                        type: string
                        description: Docker image to run the Task within. Only valid (and required) for Linux workers.

                    consumes:
                        type: object
                        description: Artifacts from other Tasks that this Task consumes, keyed by upstream Task name.
                        patternProperties:
                            ^.*$:
                                type: array
                                items:
                                    type: string
                                    description: Artifacts to download from the named upstream Task

                    workerType:
                        type: string
                        description: Worker to run this Task on
                        enum:
                            - b-linux
                            - b-macos11.0
                            - b-windows2012
                            - t-linux
                            - t-macosx11.0
                            - t-windows10
```

## Adding complex Taskcluster configuration

It will not be possible (nor desireable) to implement all possible tasks within the simplified config described above. In particular, Tasks that need custom loaders or transforms will need to be defined in the traditional way - with a custom kind that fully describes the Task, loader, and any necessary transforms. Taskgraph will be able to handle these types of Tasks referencing those found in the simplified config, and vice versa. This allows simple Tasks to be used as much as possible, and to opt-in to more complex Tasks only where needed. Here's an example of how this would work:

`.taskcluster/config.yml`:
```yaml::
baseConfig: https://some/releng/owned/thing/taskcluster.yml-v1
extraTransforms:
    - treeherder
tasks:
    build-macos:
        workerType: b-macos
        command: xcode build
        produces:
            macos-unsigned.dmg: build/macos-unsigned.dmg
            tests.tar.bz2: build/tests.tar.bz2
    test-macos:
        workerType: t-macos
        consumes:
            # This is a reference to the task above
            build-macos:
                - tests.tar.bz2
            # This is a reference to the task in `kinds/sign/kind.yml`
            sign-macos:
                - macos-signed.dmg
        command: tar -jvxf tests.tar.bz2 && run_tests.py --build macos-signed.dmg
```

`.taskcluster/kinds/sign/kind.yml`:
```yaml::
loader: taskgraph.loader.transform:loader

transforms:
    - taskgraph.transforms.signing:transforms

jobs:
    sign-macos:
        description: sign macos build
        name: signing
        worker-type: scriptworker/mozilla-1-iscript
        worker:
            implementation: iscript
            cert: dep
            upstreamArtifacts:
                # This is a reference to the task in `config.yml`
                build-macos:
                    macos-unsigned.dmg:
                        formats:
                            - mac_sign
```

## Taskcluster-Github changes

Taskcluster-Github will need to be updated to support the new `.taskcluster/config.yml` config. When it receives an event, and that file is present in the repository, it will:
* Dereference `baseConfig` to find the `.taskcluster.yml` 
* Process the `.taskcluster.yml` as usual (in most cases this will result in a decision task being created)

A new context variable, `simple_config`, will be introduced. When a `.taskcluster/config.yml` file is present in the repository this variable will be populated with an URL to it.

## Taskgraph changes

In order to support transforming simple configs into actual tasks `taskgraph` will need to provide a number of things it currently does not. Things that aren't Mozilla-specific will go in core `taskgraph`, while things that are will go in a new `mozilla_taskgraph` module. This will be similar to what we already do we modules such as `fenix_taskgraph`, except that it will be a place for storing things that are generally useful to Mozilla projects - not just a single one.

In core `taskgraph`, we will provide:
* Docker images for builds and tests on Linux (detailed in [RFC #33](https://github.com/mozilla-releng/releng-rfcs/pull/33))
* A .taskcluster.yml file that supports `push`, `pull-request`, and `action` `tasks_for`
* A `pr-complete` kind (like we use in many existing repos, to have a simple way to know when a Task Group has completed)
* A `simple_task` kind, loader, and transform that can translate tasks from `.taskcluster/config.yml` to real Tasks
    * The `simple_task` loader will work similar to the `jobs-from` mechanism used in Gecko, but source Tasks from `.taskcluster/config.yml`
    * Including sensible defaults for maximum runtimes & task priorities

In `mozilla_taskgraph`, we will provide:
* Worker definitions for available build and test machines (detailed in [RFC #33](https://github.com/mozilla-releng/releng-rfcs/pull/33))
* A `mozilla_task` transform that adds Mozilla-specific configuration and defaults to simple tasks (most notably, `trust-domain` and Treeherder metadata)

Notably, there are is no special handling for `build` and `test` Tasks - both are handled by the `simple_task` kind. Because the types of things that would normally be different between builds and tests will be defined in project repos (docker images, workers, dependencies, etc.), there is no need to have separate kinds in taskgraph.

Because we are separating out Mozilla-specific things into a separate module, we need a way to register that module before any decision task logic is run (to ensure everything inside of it is accessible when generating Tasks). This will be accomplished by adding support for a new `extra-modules` parameter to the decision task command, and the RelEng maintained `.taskcluster.yml` files will ensure `mozilla_taskgraph` is available, and passed to the decision task.

# Open Questions

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

