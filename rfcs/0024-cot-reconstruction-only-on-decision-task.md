# RFC 24 - Only Perform Chain of Trust Task-Reconstruction Once per Graph
* Comments: [#24](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/24)
* Proposed by: @mitchhentges

# Summary

Only perform "task recontruction from `.taskcluster.yml`" in the decision task, and append security
information such as `is_commit_trusted` (is commit from the source repository in the primary branch)
to the signed `chainOfTrust.json` file.
Follow-up security-sensitive tasks such as for signing or pushing to Google Play will use this
appended security information to make decisions, rather than having to rebuild the decision task from
scratch, once for each task. 

## Motivation

**We can simplify projects that run `cot`-enabled tasks**

Taskcluster allows scheduling tasks with default values. However, `cot` is not aware of these defaults.
So, we have had to add unnecessary explicit default values to our created tasks. If we no longer
do decision-task-rebuilding from follow-up tasks, we can avoid this.

**We can simplify the `cot` work that happens after the decision task**

By simplifying the work that `cot` does for follow-up tasks, we reduce bustage, 
improve maintainability, and improve performance.
 

# Details

Before running security-sensitive builds (such as signing artifacts, or pushing to Google Play),
we verify that the task group meets at least the following two criteria:
* The entity that triggered the task group (user or scheduled hook) have sufficient Taskcluster scopes
* The project revision being operated on exists in the primary repository (not a fork) on the primary
branch (e.g.: `master`).

The first criteria is checked by Taskcluster, but Chain of Trust verifies the second point by:
1. For each `cot`-enabled task in the group:
    1. Fetch the `MOBILE_HEAD_...` values from the decision task
    2. Assert that the `MOBILE_HEAD_...` values are correct by rebuilding the decision task
        1. Download `.taskcluster.yml` from the repository, branch and revision specified by `MOBILE_HEAD_...`
        2. Rebuild a `jsone` context for the task group
        3. Apply the `jsone` context to the `.taskcluster.yml` to get a decision task definition
        4. Verify the created decision task definition against the real decision task that spawn the current task
    3. Perform task-specific checks, such as verifying that the (now-trusted) `MOBILE_HEAD_...` values correspond to a
    trusted source, e.g.: `mozilla-mobile/fenix`, `master` branch
    4. ...

This RFC recommends that we keep this procedure, but we only do it in the decision task.
Then, as part of that decision task, we will put processed values (e.g.: `is_commit_trusted`) into `chainOfTrust.json`
where it can be used by follow-up tasks. So:
1. In decision task:
    1. Fetch repository values, rebuild decision task from `.taskcluster.yml` as described above
    2. Perform decision-task-specific checks
    3. Encode properties like `is_commit_trusted` into `chainOfTrust.json`
    4. Artifacts (including `chainOfTrust.json`) are signed with a trusted key
2. In follow-up tasks:
    1. Verify `chainOfTrust.json` is signed with the trusted key
    2. Get properties from `chainOfTrust.json`, then perform task-specific checks
    3. ...


# Open Questions

* Are there any negative security implications of this that have been missed? 

# Implementation

