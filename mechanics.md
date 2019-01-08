# Mechanics

So, how do I use this?

# External requests

If you are coming from outside Releng and you do not feel like you have the adequate context or expertise to make an official RFC, you may create a Github Issue instead.
A Github Issue is good for solicitating Releng help with (1) confirming the problem is unknown and not tracked and (2) forming your Issue into an official RFC proposal.

# Internal requests

## Draft phase

**Duration**: No time frame. Ideas can stay open indefinitely
**Number of people involved**: Minimum required to make a proposal. Since you do not need consensus or compromise at this stage, you can go through this phase on your own if that makes sense to do so. It's good to have an expert in the area involved or at least someone to bounce your idea off of.
**Label**: `Phase: Draft`

### Make a PR

Ideas should have
 * a pretty specific goal: a user story ("Users can..."), dependency ("This will allow ..") or a completion condition ("All services .."); and
 * some vague idea of how this might be accomplished, or a few alternatives.

If you have an idea, here's how to get started:

* Copy [`template.txt`](rfcs/template.txt) to `rfcs/xxxx-<rfc-title>.md`.
  Fill in the first few sections, but feel free to leave things TBD at this stage.
  Commit it to a branch, push, and make a pull request.

* Once you have the pull request number, modify the filename to include it (`rfcs/<number>-<rfc-title>.md`).
  Replace `<number>` in the file with the PR number as well.
  Push again.

* Label the PR `Phase: Draft`

### Open PR for comments

You can leave the PR un-assigned, or assign it to yourself if you intend to champion it.
Shop the PR around to the team and outside the team to try to get a diversity of opinions on it.
It's OK to leave ideas open indefinitely, awaiting their time.

Have others comment on the Github Pull Request.

The Github comments are intended to record the discussion on the idea, so that's the right place.

As the discussion evolves, the assignee will update the Markdown file in the PR to match.

### A fully drafted RFC is ready

When there is both a goal and solution fully formed, it's time to move to the proposal phase. Note, all open comments should have responses, even if there is not consensus or agreement.

## Proposal phase

**Duration**: ~1 week
**Number of people involved**: any person or team this may impact or who may care of this change. 
**Label**: `Phase: Proposal`

### Update the RFC

* Fill in the remainder of the template, including lots of details, based on the preceding discussion.
* Push your changes
* add the label `Phase: Proposal` to the PR.

### Involve larger audience

Now get anybody you know will care about this on-board quickly by contacting them by whatever means you want: irc, email, @ mentions should all work well.  As early as possible, get an email out to release-engineering <release-engineering@lists.mozilla.org> if this is going to be something more than a couple specific people care about. Use your best judgement.  Give a few days for opinions to filter in. Respond promptly if possible to avoid a confusing conversation but give enough time for people who are PTO or busy etc. The idea here is to work out any disagreement and come to a proposal everyone can live with, so modify the proposal as necessary using the usual git tools. Reaching consensus is unlikely, but compromise is always possible.

## Final Comments

**Duration**: ~48h
**Number of people involved**: major stakeholders in the discussion
**Label**: `Phase: Final Comment`

When the open questions have been answered and you feel that everyone is in agreement or is willing to compromise, it's time for the final-comment period.  The intent of this phase is to allow someone to speak up and say, "uh, no, that's not what I thought we decided as a group" or "I wasn't aware of this proposal, that's crazy", so notification should be distributed broadly.  The phase should last long enough for everyone to read the summary and speak up, taking into account timezones, PTO, and email backlogs - use your best judgement.  This should be at least 24 hours so that everyone has working-hours to think and respond.

Update the issue's label to `Phase: Final Comment` and send a note summarizing the proposal and indicating the duration for comments to the tools-taskcluster list, or to some other appropriate venue.

## Decided

When the final comment period has expired, if there have been no objections, mark the issue as `Phase: Decided`, file a tracker bug, update the *Implementation* section to point to it, and click "Merge". Close the PR.

**Note:** It is important to not let progress slow by diving into the implementation details as part of planning. Once the RFC has been decided, closing and merging the PR is good practice and any concerns with the implementation should be handled by the specific implementation file tracker bug, code review PR, or source of documentation.

Note that it's OK to have decided RFCs which aren't being actively worked on. However, once an RFC has been implemented, please move the RFC to the [rfcs/implemented](rfcs/implemented) directory.
