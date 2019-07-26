# RFC 30 - Allow Release Management to provide secondary signoff for the Firefox Beta channel
* Comments: [#30](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/30)
* Proposed by: @mozbhearsum

# Summary

Allow Release Management to provide secondary signoff for changes to the Firefox Beta channel in Balrog.

## Motivation

We ship Betas all the time, and Release Management often has to chase down whomever is on releaseduty to provide secondary signoff before we ship. Allowing another Release Manager to provide this signoff will reduce friction and delays.

# Details

To implement this we will create a new Role in Balrog called `releng-relman`. This Role will be assigned to members of both teams. We will updated the Firefox beta channel Required Signoffs to require signoff from 1 member of the existing `relman` group, and 1 member of the new `releng-relman` group.

As a reminder, Balrog's Signoff system was primarily designed to ensure that no single bad actor or compromised account can affect what is being shipped to users. This change does not compromise that goal.

# Open Questions

One risk that has been called out is that without RelEng forcibly in the loop, it may be easier to forget to make necessary changes that automation doesn't care of, such as setting up watershed rules. Is this an OK risk/reward for the Beta channel?

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

