# RFC 0057 - Github trust levels
* Comments: [#46](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/46)
* Proposed by: @ahal

# Summary

In Firefox-CI we use "levels" to denote trust boundaries. These boundaries and the degree
of security each receives, can vary greatly from one project to the next. While they've
worked well, they were also designed and implemented with the Firefox repo in Mercurial in
mind, and then later shoe-horned onto our Github repos.

Now that we are close to fully migrating off of Mercurial, it's time to re-define these
levels for a Github context.

## Background & Motivation

Currently there are three levels, but in Github we only use two:

* `Level 1` - All pull requests and some branches (typically used for testing)
* `Level 3` - Production branches and releases

This split does not adequately capture the trust boundaries inherent in a Github repo,
which are roughly (least to most trusted):

* Pull request from external contributor
* Pull request from a collaborator (has explicit repo access)
* Push to a non production branch
* Push to a production branch
* Release

Collapsing all these areas down to two levels (especially untrusted vs trusted PRs) opens
the door to all kinds of security bugs (e.g
[2062188](https://bugzilla.mozilla.org/show_bug.cgi?id=2062188),
[2061181](https://bugzilla.mozilla.org/show_bug.cgi?id=2061181) and
[2056955](https://bugzilla.mozilla.org/show_bug.cgi?id=2056955))

By ensuring the levels accurately reflect trust boundaries in Github, we can make
Firefox-CI more secure, and spend less time addressing security issues.

# Details

This RFC clarifies two things:

1. The definition of the term "level" itself.
2. The actual levels that will be used by Github projects.

It will additionally stop using the `public` pull request policy across all
Firefox-CI projects.

## Definition of Level

The term "level" often gets conflated. Sometimes it's meant to denote relative trust
boundaries within a project (e.g pull request vs push), but other times it's used to talk
about security guarantees (e.g firewall on the workers).

Henceforth, levels specifically refer to the relative trust boundaries within a particular
trust domain. Projects in different trust domains can have wildly different security needs
and measures, so it's not possible to meaningfully reduce a project's security standards
down to a single number. In some cases a level 1 context for a project in one trust domain
might actually be more secure than a level 3 context for a project in another!

That being said, in practice we'll continue to tie our Firefox-CI security measures
tightly to levels, and there will be many security guarantees we can make across all
projects on Github. But it's important to note the distinction that "level" refers to
trust, not security.

## Github Levels

While there are five trust boundaries identified above, there will be only three levels
defined for Github projects. They will be:

* `Level 1` - Untrusted pull requests from external contributors
* `Level 2` - Pull requests from collaborators + pushes to non-production branches
* `Level 3` - Pushes to production branches + releases

Batching trusted pull requests and pushes to non-production branches makes sense because
both require explicit repo access, and neither affects production systems.

Batching pushes to production branches and releases together makes sense because releases
deploy code from a production branch anyway, so a compromise in code there can be just
as bad as a compromise in the release pipeline. Besides in our snowman model, release
graphs rarely do additional testing than what happened on the push (with some exceptions).

This strikes a balance between convenience (having five levels means more pools and busy
work) and security (it addresses the main concern of trusted vs untrusted PRs).

## Public Pull Request Policy

Taskcluster provides a `public` pull request policy. This means that all PRs to a project
get the same role, regardless of whether it comes from a collaborator or not. This mode
can cause confusion, since collaborator and contributor PRs all have the same scopes, what
level should they be?

This policy is a weaker security stance than the alternative `public_restricted` stance,
and doesn't provide any benefit. So to help reduce confusion, all Firefox-CI repositories
will be migrated from the `public` to `public_restricted` policy. This way collaborator
pull requests will _always_ be `level 2` and external pull requests will _always_ be
`level 1`.

## Scope

This RFC is explicitly not concerned with the security measures that are associated with
each level. For example, we likely want to have a rule that all level 3 branches have
branch protections enabled.

But this RFC is specifically focused on trust rather than specific security measures.

# Open Questions

* Should we distinguish between collaborator permissions, e.g triage vs write vs admin?
* Do we need to carve out new L2 pools on limited Mac hardware, or do we leave those on
  L1/L3 as is?
* Should we have additional security guarantees for L2? (CoT, livelogs, worker access)
  * Not part of this RFC, but worth a follow-up after implementation
* What level are Gecko project branches?
  * Since "levels" are relative to each project, I believe project branches should follow the same
    L1/L2/L3 splits as everything else.
  * But I also consider this question out of scope for this RFC.

# Implementation

Tracking bug: TBD
