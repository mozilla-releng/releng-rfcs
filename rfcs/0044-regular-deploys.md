# RFC 44 - Scheduled Updates & Deploys of Balrog, Ship It, and Scriptworkers
* Comments: [#44](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/43)
* Proposed by: @bhearsum

# Summary

We should regularly update and deploy our most crucial services to product: Balrog, Ship It and Scriptworkers.

## Motivation

As these projects have matured and stabilized the time between deploys has lengthened. Not exercising our deployment pipeline regularly increases the risk that it will be broken at an inopportune time (such as when deploying an urgent fix). Most recently, we had an example of [Balrog's kubernetes configs needing a bump](https://github.com/mozilla-services/cloudops-infra/pull/5041) before deploys could successfully complete.

Additionally, we have not been keeping up on dependency updates in either of these projects. This both increases the risk of security issues cropping up, and the risk of getting far enough out of date that we can no longer take _any_ dependency upgrades without major headaches or overhaul.

# Details

## Backend Dependencies Only

For the time being, we will only updated backend dependencies as part of these. We have much more limited frontend knowledge, and zero tests for frontend - so regular updating of these dependencies poses a much higher risk of bustage, and when it happens, will be more difficult to isolate and fix.

## Process Changes

Few changes will be needed to implement this proposal. We already have tools and processes for updating dependencies and deploying new versions - the main thing needed will be reminders for those on releaseduty. To this end, we will add two new Slack notifications targeting @releng:

1) A reminder 2 days before the expected deploy date to update the dependencies and deploy to stage. This will give us time to sanity check and ideally fix any issues.
2) A reminder on deploy day to perform the production deployment.

The other change we will make as part of this is to ensure that RelEng can [complete Balrog deployments without the involvement of the SRE team](https://mozilla-hub.atlassian.net/browse/SVCSE-1466). (This is not strictly necessary to get where we want, but it avoids extra busy work for SREs, and moves in the direction that they would like to go anyways.)

## Schedule

We will perform the production deployments every other Thursday. To reduce risk, we will schedule these opposite the existing regular Taskcluster deployments. (That is to say: one week will be RelEng services, the next will be Taskcluster, ad infinitum.) Staging will happen two days prior, on the Tuesday.

# Open Questions

None

# Implementation

Slack reminders have been added in #releng-notifications
