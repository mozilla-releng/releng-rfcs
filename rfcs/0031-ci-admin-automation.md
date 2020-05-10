# RFC 31 - Automate running ci-admin on push. 
* Comments: [#31](https://github.com/mozilla-releng/releng-rfcs/issues/31)
* Proposed by: [@tomprince](https://github.com/tomprince)

# Summary

Automate running ci-admin on push by running it in cloudops' jenkins.

## Motivation

- Running ci-admin manually adds extra friction to the process of deploying changes,
  and can cause confusion when a change is not deployed after landing (either the person 
  landing not running or not being able to run ci-admin).
- Running ci-admin manually requires that people have sufficient scopes to apply
  changes. This means that anybody that can run ci-admin has enough scopes run
  tasks on level-3 workers and modify level-3 indexes. This makes it harder to
  reason about the security properties of the system, as we need to take into
  account people running tasks on those worker directly.

# Details

- The commit access of the [ci-admin](https://hg.mozilla.org/ci/ci-admin) and
[ci-configuration](https://hg.mozilla.org/ci/ci-configuration) repositories will
be restricted to a new scm group (`scm_firefox_ci`) and will only be able to be
pushed to via lando.

  - There will be an escape hatch of putting `MANUAL PUSH` in the commit
    message, along with a reason, that will trigger secops' audit pipeline.

- The scopes that releng and relops currently have will be restricted to those
  they need for day-to-day work, and the scopes to run ci-admin will be
  restricted to the root scope that lives in cloudops' infrastructure.

- A push to ci-admin or ci-configuration will trigger a job in cloudops'
  jenkins instance that will run ci-admin with the taskcluster root client,
  after having run ci-admin check.

# Open Questions

- What scopes do releng and relops need for day-to-day work? Since we currently
  have root-equivalent scopes, we have not needed to figure out what scopes we
  need otherwise.
- We need to run `ci-admin` when `.taskcluster.yml` is updated in various repos
  (to update action hooks). We'll need a way to trigger this, or move away from
  embedding `.taskcluster.yml` in the hooks.

# Implementation

* [Bug 1619470](https://bugzilla.mozilla.org/show_bug.cgi?id=1619470): Run ci-admin in cloudops's jenkins instance
* [Bug 1632009](https://bugzilla.mozilla.org/show_bug.cgi?id=1632009): Manage taskcluster clients via ci-admin
* [Bug 1645168](https://bugzilla.mozilla.org/show_bug.cgi?id=1645168): Restrict pushes to ci-config and ci-admin repos to via lando
* [Bug 1645170](https://bugzilla.mozilla.org/show_bug.cgi?id=1645170): Restrict pushes to ci-config and ci-admin repos to the scm_firefoxci group.
* [Bug 1645178](https://bugzilla.mozilla.org/show_bug.cgi?id=1645178): Please monitor pushes to ci/ci-configuration and ci/ci-admin repositories with `MANUAL PUSH` in the commit message.

