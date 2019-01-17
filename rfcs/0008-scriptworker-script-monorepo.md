# RFC 0008 - Scriptworker \*script Monorepo
* Comments: [#8](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/8)
* Proposed by: @escapewindow

# Summary

We should put all of our releng-maintained production scriptworker \*scripts into a single repo, so we can improve discoverability, keep \*script behaviors consistent, and make it easier to make \*script-wide changes.

## Motivation

We have 10 \*scripts in puppet currently, and are looking at adding more. Currently that means we have 10 github repos, plus scriptworker itself for `scriptworker.client`, which most \*scripts import for shared behaviors.

This proves challenging for:

* discoverability. If you want to find all of the scriptworker \*scripts, you can browse the mozilla-releng organization for any repos that seem to match the right naming scheme. As the number of repos increase, or if a \*script is outside the mozilla-releng org for any reason, it will get worse.
* consistency in \*script behavior. Outside of recommended behaviors and the shared code in `scriptworker.client`, the separate repos lend themselves to divergence of behavior.
* consistency in github automation. Similar to the \*script behavior, the travis+pyup+code coverage automation configurations have already diverged.
* consistency in \*script deployment. With each maintainer defining their own behaviors, we've deployed some \*scripts to pypi, added towncrier to some.
* making changes across \*scripts. We've had a few instances of this in recent memory, where someone has to create PRs across 10+ repos.

Moving to a monorepo for \*scripts improves all of the above:

* discoverability: if you find the monorepo, you've found all of the \*scripts.
* consistency in \*script behavior: it's possible to inspect and modify \*scripts' behavior in a single place.
* consistency in github automation: with a single `.travis.yml` and/or `.taskcluster.yml`, and a single repo to point other services at (e.g. code coverage, pyup).
* consistency in \*script deployment: we're currently planning on converging on nix, but even nix-based solutions could diverge over 10+ repos. The monorepo allows us to configure them all from the same place.
* making changes across \*scripts: even if we want to make a sweeping change, we could find the relevant scripts in the same repo.

We are still able to make exceptions for behavior in \*scripts when needed. For example, we could have exceptions to allow balrogscript to run on py2, until we address that issue. We should be able to temporarily branch and/or keep a \*script on an older codebase (e.g. by not deploying the latest to production) if needed.

(Coop tweeted a [thread about taskcluster going to a monorepo](https://mobile.twitter.com/ccooper/status/1081264280429871104). We have a similar situation: small code size, small team size, and we're also going to be handing off operational responsibilities to cloudops.)

# Details

We create a github repo, `mozilla-releng/scriptworker-scripts`. We move the code from `scriptworker.client` into a `scriptworker-client` subdirectory, with its own `setup.py`. Tests will go under each subdirectly, e.g. `scriptworker-client/tests/` or `signingscript/tests`. We can move each production-ready, releng-maintained \*script into the monorepo over time.

Each \*script will have its own `setup.py`.

Initially, we can cover automated testing via a `.travis.yml` that covers the entire repo, but this will be complex enough that we will want a `.taskcluster.yml` to test each \*script concurrently, and potentially build wheels and docker images in automation.

Any change in shared behavior should span all \*scripts as applicable.

We can do things like pyup dependency pinning across the repo like we do with puppet.

We need to have ways of excluding or special-casing \*scripts. `balrogscript` is py2; the upcoming `applescript` will be targeted towards running on mac hosts instead of deployed through kubernetes.

# Open Questions

We need a way to handle applescript and balrogscript differently from the rest of the \*scripts.

We haven't touched binary transparency in a long time; perhaps we should call this not-yet-production-ready and wait to move it?

Do we call it mozilla-releng/scriptworker-scripts ?

Specific details about directory layouts, tox.ini per subdirectory or top level?, pyup pinning standards? nix?, code coverage service?, but I imagine we will decide on something good, and have the capability of changing it in the future without having to touch 10+ repos for consistency.

# Implementation

