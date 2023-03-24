# RFC 0042 - Decommission RelEng JIRA Project in favor or Bugzilla's product "Release Engineering"

* Comments: [#42](https://github.com/mozilla-releng/releng-rfcs/pull/42)
* Proposed by: [@JohanLorenzo](https://github.com/JohanLorenzo)

## Summary

Approximately 3 years after filing [RelEng's first Jira ticket](https://mozilla-hub.atlassian.net/browse/RELENG-2) and 1000+ tickets later, the Release Engineering team is still wondering which issue tracker to use. This causes confusion regarding where to put daily engineering work. Everyone on the team agrees we need clarity on this.

After reviewing the original intent, the team's current practice, and what leadership expects from us, we are going to decomission [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG).

## Motivation

### Why did RelEng go to Jira in the first place?

Like said above, RelEng started to use Jira 3 years ago. This was an immediate response to [August 2020's layoff](https://blog.mozilla.org/en/mozilla/changing-world-changing-mozilla/). At that time, we wanted to make sure RelEng's work was visible to leadership. We filed our biggest projects on Jira, leaving Bugzilla in second place.

### What's changed since we made that decision?

Since then, the team has evolved. New members have joined the team. Sometimes new members were old-guard Mozillians, sometimes they were new Mozillians. No matter the profile, it remains unclear where our daily work needs to be tracked. This results in a split-brain situation where some of knowledge is on Bugzilla, some is on Jira.

Since then, the need for Bugzilla has come back. We cannot be entirely on Jira because any patch that lands on [mozilla-central](https://hg.mozilla.org/mozilla-central/) must have a Bugzilla bug number. Some teams, like the Android ones [moved their issue tracking back to Bugzilla](https://github.com/mozilla-mobile/android-components/issues/12751).

Since then, we have learned that any Jira tickets filed outside of some specific Jira projects remain invisible to leadership. To be more explicit, the [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG) project is invisible to upper management.

Since then, leadership [clarified the intent](https://docs.google.com/document/d/1JMf6kEZpzetkEC3mOb80KYSY0NrKONU-Rs0oNotgAhc/edit) of making Jira available at Mozilla:

> A key benefit of having Jira connected but have different and complementary functionality, is solving the problem of communication with cross functional disciplines, leadership and scenario planning. Previously, this work might have been handled in spreadsheets, for example.

## Details

### Scope of this RFC

#### We are not able to chose one single solution

For sure, we must use Bugzilla for any work related to Gecko. Leadership needs to be aware of what RelEng does through Jira.

#### GitHub issues are a separate conversation

GitHub issues are another bug tracking system we use. It's another split-brain situation. We agree it's conversation for later. Therefore, GitHub issues won't be part of the equation.

#### Short-term planning is also a separate conversation

In this RFC, we are willing to solve one problem: where should daily engineering work should be filed on? While the answer has an impact on the way RelEng tracks and plans short-term work, this is not the only parameter to take into account.

For context, the Release Engineering tried Scrum/Kanban planning on [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG). We stopped about a year and a half ago. Therefore, if we want to adjust our current way of planning short-term, it's going to be a bigger conversation which will involve:
 * current habits,
 * defining a process,
 * agreeing on the way we all use tools,
 * being in line with what other teams do.

As a result, this RFC won't be on short-term planning.

### Proposal

#### Jira for high-level project planning

Mozilla has [a process](https://docs.google.com/document/d/1BA1v2xiDF1IoqbyfY3hWlXc_MRsI02hYs2rslmkK3gM/edit#) to define which projects are more important than others. In the case of RelEng's organization, this happens within the [FFXP](https://mozilla-hub.atlassian.net/browse/FFXP) project on Jira. It’s typically the job of a manager to create Jira's epics there. The content of these epics are centered around 2 main fields: a high level description of what we plan to do and "why now?".


#### Bugzilla for daily engineering work and implementation details

Being open is part of [Mozilla's identity](https://www.mozilla.org/en-US/about/manifesto/). Our Jira instance is not open. Therefore, it should not be the source of truth, especially regarding implementation details.

Moreover, Mozilla has heavily invested in tools what work in conjunction with Bugzilla:
 * Phabricator
 * Lando
 * hg.mozilla.org
 * github.com (PRs get automatically attached to bugs)
 * Element (bots and link previews)
 * Slack (link previews only)

Mozilla doesn't have the same level of quality and support when it comes to Jira.

### Consequences

#### [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG) will be decommissioned

Considering all of the above, which includes the original intent when we created [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG), we shouldn't do daily engineering work on Jira anymore. Hence, [RELENG](https://mozilla-hub.atlassian.net/browse/RELENG) will be retired in favor of Bugzilla bugs under the [Release Engineering](https://bugzilla.mozilla.org/describecomponents.cgi?product=Release%20Engineering) product.


#### Jira will still be occasionally used by engineers

[FFXP](https://mozilla-hub.atlassian.net/browse/FFXP) is one place but RelEng interacts with teams which are solely on Jira. We will comply with what these teams chose.


#### There won't be any synchronization between Jira and Bugzilla.

[FFXP](https://mozilla-hub.atlassian.net/browse/FFXP) is for mid-to-long term planning. This Jira project doesn't have any automation in place. That shouldn't be a concern because epics on [FFXP](https://mozilla-hub.atlassian.net/browse/FFXP) don't change as often as Bugzilla bugs.


## Open Questions

* Do we want to backport the ~1000 tickets to Bugzilla?
* Do we want to import the currently open tickets to Bugzilla?

## Implementation

N/A
