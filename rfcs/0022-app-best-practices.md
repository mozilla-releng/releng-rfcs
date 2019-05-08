# RFC 22 - Best Practices for RelEng Applications
* Comments: [#22](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @rail, @mozbhearsum

# Summary

Use standard and well known tools & processes for development, testing, building, and deploying RelEng applications. Specifically:
* Use Docker & Docker Compose for running applications and tests locally
* Use language appropriate tools for testing and dependency management
    * For Python, this means `pip-compile-multi`, `tox`, and Pyup.io
    * For Javascript, this means `yarn`, `neutrino`, `jest`, and Greenkeeper.io
* Use Taskcluster to run tests and builds in CI
* Maintain appropriate deployed environments that allow testing to be done before pushing new code to production.

## Motivation

Release Engineering has evolved a large set of applications over a number of years. Each application has created its own way of running tests, doing local development, building, managing dependencies, etc. Some applications also use tools that are not well known (eg: Nix, `lineman`) or custom solutions to problems that could be solved by standard tools (eg: Balrog's `run-tests.sh`). This inconsistency and use of unfamiliar tools make it more difficult to ramp up on new projects, switch between projects, and generally reduce our development velocity. Standardizing on a set of well known tools & processes will allow us to move faster and onboard new people (both employees and volunteers) more quickly.

# Details

The below are best practices. It is strongly recommended that you stick to them, but there are cases where it may make sense to do something differently. For example: `iscriptworker` runs on OS X, and thus does not make sense to build or deploy with Docker.

## Development

Docker and Docker Compose should be used to build, run, and test applications locally. This ensures that all developers, CI, and deployed environments are as identical as is reasonable. Using Docker means that we will explicitly not be pinning system level packages anymore. This is an explicit trade-off that we are making -- the cost/complexity of doing so is not worth the small benefits we get from it.

Python applications should use `python:3` as their base image. Javascript applications should use `node:current` as their base image.

## Dependency Management

Python projects should specify their dependencies in three files (as is standard for `pip-compile-multi`):
* `requirements/base.in` will contain direct dependencies required for running the application
* `requirements/test.in` will contain direct dependencies required for running tests, and a reference to the `base` requirements
* `requirements/local.in` will contain direct dependencies required for local development (at minimum, `tox` and `pip-compile-multi`), and references to the `base` and `test` requirements

The `pip-compile-multi` documentation has [additional details and reasoning](https://github.com/peterdemin/pip-compile-multi#managing-dependency-versions-in-multiple-environments) behind this split for requirements files.

When any of these files change, `pip-compile-multi` will be used to generate the full list of pinned dependencies. Pyup.io will be used to keep them up-to-date. It is recommended that Pyup be configured to use batch mode (instead of PR-per-dependency) to avoid issues where multiple dependencies need to be updated at the same time.

Javascript projects will specify their dependencies in `package.json`. As is the usual for Javascript projects, direct dependencies will be listed in `dependencies` and direct test and local development dependencies will be listed in `devDependencies`. When either of this lists change, `yarn lock` will be used to generate the full list of pinned dependencies. Greenkeeper.io will be used to keep them up-to-date.

## Testing

### Unit Tests

All testing should be done within Docker container. You may also run tests directly on your host machine, but we should not spend time optimizing or fixing failures that don't occur in Docker.

Python projects should wrap Docker with `tox` while Javascript projects should wrap Docker with `yarn`. These wrappers will build the necessary Docker image, and then run the tests within it. This is to maintain a familiar, well-known entrypoint for running tests, while ensuring they run in a consistent way for all developers.

All CI should be done with Taskcluster. CI tests should be run in images built from the same Dockerfiles used when running tests locally.

## Builds

Python projects should be built Docker. Production and local development have typically slightly different requirements when it comes to their Docker images. To accommodate this while minimizing the possibility of difference between them, we will use a multistage Dockerfile to allow them to live nearby. For example, here is a Dockerfile that can build both the production and local development versions of a simple Python project:
```
ARG PYTHON_VERSION=3.7
FROM python:${PYTHON_VERSION}-slim as production

RUN apt-get update && apt-get upgrade -y

RUN python -m venv /venv
COPY ./ ./src
RUN cd /src && /venv/bin/pip install -r requirements.txt


FROM python:${PYTHON_VERSION} as testing

COPY --from=production /venv /venv
COPY ./ ./src
RUN cd /src && /venv/bin/pip install -r requirements-dev.txt
```

Javascript projects will be built with `yarn build`.

## Environments

Every application should have a `production` environment, as well as appropriate, deployed non-production environments to ensure that code can be effectively tested before pushing to `production`. Unless you have a compelling reason to do something different, this means a `stage` environment.

### Production

Production builds should happen in response to Github `release` events.

Python projects should push both a `${project}_production` and `${project}_${hash}` tag to Dockerhub. If the project autodeploys to `production`, it should do in response to the `${project}_production` tag. If it manually deploys, a bug should need to be filed that references the `${project}_${hash}` tag.

Javascript projects should publish a built version of the app as a Taskcluster artifact. If the project autodeploys to `production`, it should also publish to S3. If it manually deploys, a bug will need to be filed that references the built app.

### Stage & Development

Stage builds should happen in response to push events on the `master` branch. Once a build is completed, it should automatically deploy to the `stage` environment. For Python projects, this means pushing a `${project}_stage` tag to Dockerhub. For Javascript projects, this means publishing to S3.

As much as possible, development environments should be brought up on an ad-hoc basis. In many cases a development environment running on your own laptop should be sufficient. Running development environments on AWS or GCE instances is also acceptable.

In some cases (most notably, scriptworkers that need routes to autograph or other external services), ad-hoc development environments are not realistic. In these cases, it is acceptable to deploy your own code to a `stage` environment.

## Dependencies between applications

Any application that depends on another application should use the same environment. Eg: Ship It `stage` should use Balrog `stage`, not Balrog `production`.

Dependencies on external applications are a little more ambiguous, as not all external dependencies have a matching set of environments. Use your best judgement here. Eg: don't have production services depend on external staging services if you can help it (or vice versa).

## Notes on `mozilla-releng/services`

`mozilla-releng/services` is where a large number of our applications live. They are currently built and managed with Nix and a special command called `please`, which wraps Docker, `tox`, and other tools. Upon acceptance of this RFC these will be considered deprecated, and work towards removing them fully.

Release Management has already made the decision to remove their projects from this repository. Release Engineering projects may continue to live in this repository, or may be moved elsewhere on a case-by-case basis.

Shared code in the `lib` directory may be duplicated to any projects that require it.

# Open Questions

# Implementation
