# Mechanics

So, how do I use this?

## Identify phase

At this stage, you are trying to identify a goal and potential solution. These are defined as:

* a pretty specific goal: a user story ("Users can..."), dependency ("This will allow ..") or a completion condition ("All services .."); and
* some vague idea of how this might be accomplished, or a few alternatives.

If already have this, feel free to skip to the [idea phase](#idea-phase)

Otherwise, start by creating a Github Issue. Issues are good for solicitating help with:

1. confirming your thoughts are not based off missing context or knowledge
2. forming those thoughts into an idea.

After creating the issue, feel free to leave it open, iterate on it more, or ping others for input.

## Idea phase

**Duration**: No time frame. Ideas can stay open indefinitely

**Number of people involved**: Minimum required to make a proposal and expect the PR to be signed off. It's good to have domain experts or stakeholders involved.

In general, the solution proposed in the next step should have an expectation of being signed off, though specific details may change. An open-ended discussion starting from the problem statements may be more productive, and allow for people to reach common ground earlier, than starting from a potentially controversial solution that hasn't been discussed beforehand. If the various stakeholders agree that a fully fleshed out proposal would be preferable, even before any consensus, this rule of thumb may be waived.

This discussion can happen in the issue created in the identify phase, or on another platform. If the discussion happens on another platform, and if this idea proceeds to the next phase, it's a good idea to encapsulate that discussion in the issue or PR.

**PR Label**: `Phase: Draft`

### Make a PR

Once you have a goal and potential solution that is expected to pass (though specific details may change), you can create a PR.

* Copy [`template.md`](rfcs/template.md) to `rfcs/xxxx-<rfc-title>.md`.
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

When the template RFC can be fully filled out and all comments have been considered, it's time to move to the proposal phase. Note, all open comments merely require responses, they do not require consensus or agreement on all the details yet.

## Proposal phase

**Duration**: ~1 week

**Number of people involved**: any person or team this may impact or who may care of this change. 

**PR Label**: `Phase: Proposal`

### Update the RFC

Fill in the remainder of the template, including lots of details, based on the preceding discussion. Push your changes. Add the label `Phase: Proposal` to the PR.

### Involve larger audience

Now is the time to loop in anybody you know you think will care about this RFC by contacting them by whatever means you want: irc, email, @ mentions should all work well. Get an email out to release-engineering <release-engineering@lists.mozilla.org> if this is going to be something more than a couple specific people care about. Use your best judgement.  Give a few days for opinions to filter in. Respond promptly if possible to avoid a confusing conversation but give enough time for people who are PTO or busy etc. The idea here is to work out any disagreement and come to a proposal everyone can live with, so modify the proposal as necessary using the usual git tools. Reaching consensus is unlikely, but compromise is always possible.

## Final comments phase

**Duration**: ~48h

**Number of people involved**: major stakeholders in the discussion

**PR Label**: `Phase: Final Comment`

When the open questions have been answered and you feel that everyone is in agreement or is willing to compromise, it's time for the final-comment period.  The intent of this phase is to allow someone to speak up and say, "Uh, no, that's not what I thought we decided as a group" or "I wasn't aware of this proposal, that's crazy", so notification should be distributed broadly.  The phase should last long enough for everyone to read the summary and speak up, taking into account timezones, PTO, and email backlogs - use your best judgement.  This should be at least 24 hours so that everyone has working-hours to think and respond.

Update the issue's label to `Phase: Final Comment` and send a note summarizing the proposal and indicating the duration for comments to the tools-taskcluster list, or to some other appropriate venue.

## Decided phase

When the final comment period has expired, if there have been no objections, mark the issue as `Phase: Decided`, file a tracker bug, update the *Implementation* section to point to it, and click `Merge`. Close the PR.

**Note:** It is important to not let progress slow by spending cycles on the implementation details as part of the RFC and planning process. Once the RFC has been decided, closing and merging the PR is good practice and any concerns with the implementation should be handled by the specific implementation file tracker bug, code review PR, or source of documentation change.

Note that it's OK to have decided RFCs which aren't being actively worked on. However, once an RFC has been implemented, please move the RFC to the [rfcs/implemented](rfcs/implemented) directory.
