# RFC 0050 - Updated System Addons release process
* Proposed by: @ahal

# Summary

The system addons release process was designed for a different time. It
involves people who don't need to be involved, and omits people who should be.

This RFC aims to streamline the process such that we can both reduce friction
and increase security.

## Motivation

System addons have largely gone unused over the past several years, but
recently activity is picking up. They are actively being used for webcompat
interventions, which can sometimes see multiple releases a month.

They are also planned to be used by the new tab team and others.

## Out of Scope

There is an adjacent question around where System Addons live. Currently their
canonical location is in the `mozilla-extensions` Github org, though some teams
are landing code in `mozilla-central` and then syncing it over. As annoying as
this is, this question is considered out of scope for this RFC. I'd like to
focus solely on the release mechanics and leave this issue as follow-up work.

# Details

## Repository

The process starts with protecting the repository source. We need to guarantee:

- Only authorized users can push changes.
- That all code changes are properly reviewed.

To this end, all teams working on system addons must create a team in the
`mozilla-extensions` org. This team should have write access to their add-on
repo and the following branch protections must be applied at a minimum:

1. Require signed commits
2. Require a pull request before merging
   - Dismiss stale pull request approvals when new commits are pushed
   - Require approval of the most recent reviewable push

3. Block force pushes

## Shipit

### Auth

By default, Releng will have the ability to sign off on all System Addon
phases. This is so we can ensure the ability to ship them in an emergency.

But for long lived system addons that are actively being maintained, it will be
required to create an LDAP group *per System Addon*. This group should contain
members of the team working on the System Addon. Ideally there should be at
least 3-4 people in this group as dual signoff will be required and this helps
account for timezones and folks being on PTO. Membership in this group grants
the ability to create and cancel releases, as well as schedule and sign off on
all phases of the release. As noted above, Releng will be able to sign off in
the event there aren't enough developers around, but this should only be done
in an emergency, and after due diligence has been undertaken to ensure the
request is legitimate.

For System Addons that are one-off / short lived, an LDAP group is not
necessary and Releng signoff will be sufficient. In this event, the request
should come out of an active incident channel.

### Phases

1. Build
   - Builds the addon
   - No signoffs required

2. Ship
   - Release signs the build, pushes to archive.m.o, creates a release in Balrog
   - Two signoffs from System Addon specific LDAP group required

Note: The ship phase does not actually ship the system addon to users, this is
handled in Balrog where the release is handed off to Releng.

## Balrog

This RFC recommends no changes to the process within Balrog, but they are listed
anyway for completeness:

1. Upon request, Releng creates the appropriate superblob and sets of the rule
   on the desired `<channel>-sysaddons` channel(s).
2. QA performs update testing to verify everything is working as expected.
3. Upon request, Releng duplicates the appropriate rule(s) over to the
   `<channel>` channel(s).
   - This requires 1 releng signoff (to validate the rules) and 1 relman
      signoff (to greenlight the timing)

# Open Questions

1. Should Releng have the ability to do Shipit signoffs in the event of an
   emergency? 
2. Is "Ship" the right name for the phase in Shipit? It's shipped in the sense
   that it is release signed and exists on archive.m.o, though not shipped to
   users.
3. Should there be two releng signoffs in Balrog? Creating superblobs and
   shipping them in the desired places can be tricky, so maybe we should
   prevent the ability of someone signing off on their own changes.

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* https://bugzilla.mozilla.org/show_bug.cgi?id=1951760
* <...>

