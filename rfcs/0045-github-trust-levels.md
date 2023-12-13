# RFC 0045 - Github Trust Levels
* Proposed by: @ahal

# Summary

In Firefox-CI there are currently three trust levels (abbreviated L1, L2 and
L3). These were designed for Gecko, but when we started supporting Github, we
shoehorned the level system on. Currently for a "level 3" project:

1. L1 is used on pull requests
2. L2 is unused
3. L3 is used for pushes

However, it's also possible for a project to be configured as a "level 1" project.
In these projects, both pull requests and pushes use L1 and L3 is unused.

This RFC proposes a redefinition of the Github trust levels, as well as some changes to
ci-config to support it. First, trust levels on Github will be defined as follows:

1. L1 is used for public (e.g untrusted) pull requests.
2. L2 is used for trusted pull requests (e.g with `collaborator` or `public_restricted` policies), as well as
   pushes to unprotected or non-release branches.
3. L3 is used for pushes to protected or release branches.

In ci-configuration, Github projects will no longer be configured with a
project wide level. Instead, pushes will use L2 by default and select branches
will be explicitly listed as L3 as necessary.


## Motivation

This change will help solve three problems:

1. Lack of separation between pushes and pull requests on level 1 projects.

   Currently PRs and pushes share the same secrets, caches and more for level 1
   projects. While these projects are not a security risk, this still
   introduces potential for footguns and makes it difficult to implement
   certain features properly. Caching in particular suffers.

2. Lack of separation between pull request policies.

   There's also no way to differentiate between trusted and untrusted
   pull-requests via the level system. E.g, they both share access to the same
   secret paths, so care needs to be taken not to accidentally expose a secret
   to the public. Having these on separate levels reduces the risk of errors
   in ci-config.

3. Gecko migration to Github.

   Gecko will be moving to Github sometime in 2024, which means we need to
   figure out how to deal with the project twigs (e.g, oak, maple, etc). We
   could explicitly fork each one and list them all out as a separate level 2
   project (like we do now). But it would be much nicer if these could be
   replaced by branches on the main repo. Supporting this will require allowing
   different branches to have different levels.


# Details

<how will this be implemented? what does it depend on? what are the
compatibility concerns?>

# Open Questions

<what isn't decided yet? remove this section when it is empty, and then go to
the final comment phase>

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

