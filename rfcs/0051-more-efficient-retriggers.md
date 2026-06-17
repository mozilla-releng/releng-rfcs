# RFC 0051: Optimize Taskcluster retrigger to avoid Action tasks congestion

* Comments: [releng-rfcs#51](https://github.com/mozilla-releng/releng-rfcs/pull/51)
* Proposed by: [@JohanLorenzo](https://github.com/JohanLorenzo)

## Summary

Introduce a `retrigger-method` attribute to allow taskgraph to determine at generation time whether a task should be retriggered via action task or direct API call. This eliminates unnecessary action tasks for leaf test tasks, preventing congestion on the decision worker pool.

## Motivation

### What caused the initial investigation

On October 3, 2025, a developer retriggered 1000+ test tasks on [this try push](https://treeherder.mozilla.org/jobs?repo=try&revision=bba54053fd29ee9e56bf341ff8fbf0450c26528d). This created a large number of action tasks that congested the `gecko-1/decision-gcp` worker pool. As a result, [this decision task](https://firefox-ci-tc.services.mozilla.com/tasks/IixqrE_-SPaVaWCsn4jkhA) waited **33 minutes** before starting.

The immediate workaround was [to increase the worker pool capacity](https://github.com/mozilla-releng/fxci-config/pull/570). While this mitigated the congestion, it doesn't address the root cause.

### The broader issue

From [the state of try](https://sql.telemetry.mozilla.org/dashboard/decision-action-tasks?p_project=%5B%22try%22%5D&p_start_date=2025-01-01) between January and October 2025, we can conclude:
  1. Retrigger action tasks are the **#1 action task type**, regularly matching or exceeding decision task volume.
  2. The vast majority retrigger **test tasks** using the default `times: 1` (single retrigger).
  3. Each retrigger action task takes 1-6 minutes to complete (typically 1 minute, but regularly 5-6 minutes)
  4. The queueing problem is correlated with retrigger Action task volume
  5. Half the time, the decision and action tasks don't start immediately and can take 1+ minutes.

**In the vast majority of cases, retriggers don't need action tasks and can be retriggered directly. Not requiring a Retrigger Action task speeds up the action of retriggering a task while not clogging Decision tasks**.

### How Retriggers Currently Work

Taskcluster has two retrigger paths (both create NEW tasks with new taskIds):

1. **Action-based**: [Triggers a hook](https://github.com/taskcluster/taskgraph/blob/de7ca2bd8a164ec42205964fd3e52863e95c1bb9/src/taskgraph/actions/registry.py#L226) → spawns [action task](https://github.com/taskcluster/taskgraph/blob/de7ca2bd8a164ec42205964fd3e52863e95c1bb9/src/taskgraph/actions/retrigger.py#L255-L294) on decision pool → action task creates retriggered task. This approach maintains Chain of Trust by linking retriggered tasks back to the decision task through the action task.
2. **Direct**: [Directly creates](https://github.com/taskcluster/taskcluster/blob/c76db520701a19cc49ded480ea3d0cb7378f252b/ui/src/views/Tasks/ViewTask/index.jsx#L600) retriggered task via `createTask` API (no action task, no CoT link to decision task)

**When each is used:**

- **Taskcluster UI**: Checks if [there are actions tasks define](https://github.com/taskcluster/taskcluster/blob/c76db520701a19cc49ded480ea3d0cb7378f252b/ui/src/views/Tasks/ViewTask/index.jsx#L151-L182). If match → action-based. If no match → direct.

- **Treeherder**: [Always uses actions](https://github.com/mozilla/treeherder/blob/e5a02d39e70ab237bb37c3f497c994b9421f5e83/ui/models/job.js#L114-L129) (calls `retrigger-multiple` or `add-new-jobs`)

Test tasks have [`retrigger: true` attribute](https://github.com/mozilla-firefox/firefox/blob/644f0db17749554fe23a45b43e77e61f42acdfd9/taskcluster/kinds/test/kind.yml#L44), which [matches the retrigger action](https://github.com/taskcluster/taskgraph/blob/de7ca2bd8a164ec42205964fd3e52863e95c1bb9/src/taskgraph/actions/retrigger.py#L80), so they always use the action-based path.


### How did we end up with these 2 mechanisms?

#### Before Action tasks (pre-2016)

Mozilla used an out-of-tree service called [pulse_actions](https://github.com/mozilla/pulse_actions) which
- Listened to pulse messages from Treeherder when users requested job scheduling
- For Taskcluster jobs: [Built and scheduled task graphs directly](https://github.com/mozilla/pulse_actions/blob/c7883d32a9fccda06047b6fad6b0e86ddae78271/pulse_actions/handlers/treeherder_add_new_jobs.py).
- For Buildbot jobs: Triggered jobs via Buildapi

Meanwhile, the Taskcluster UI only had [a direct retrigger button]((https://github.com/taskcluster/taskcluster-tools/blob/a37a7ca93de30c5153357828f091a143b6897bda/src/lib/ui/RetriggerMenuItem.jsx#L41-L43)).

#### Action Tasks are introduced (July 2016)

The [first action task](https://bugzilla.mozilla.org/show_bug.cgi?id=1281062) was created by introducing [action.yml](https://hg.mozilla.org/mozilla-central/rev/d223b3cdee66) to the Firefox repository. This enabled scheduling [new jobs on try pushes](https://bugzilla.mozilla.org/show_bug.cgi?id=1284911) by moving the scheduling logic in-tree. `pulse_actions` was updated [to trigger action tasks](https://github.com/mozilla/pulse_actions/pull/82) instead of directly scheduling Taskcluster task graphs.

#### The first Retrigger Action task (July - November 2017)

A [retrigger action](https://bugzilla.mozilla.org/show_bug.cgi?id=1380454) was [added](https://hg.mozilla.org/mozilla-central/rev/ec7291c0411c). A month later, Chain of Trust verification was [added to action tasks](https://hg.mozilla.org/mozilla-central/rev/03e6ddd50880). Per [the bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1393277): "we need to be able to retrigger failed tasks without breaking CoT." This was a new requirement, not the original motivation for action tasks. In November 2017, `pulse_actions` was [shut down](https://bugzilla.mozilla.org/show_bug.cgi?id=1379172).

#### UI Consolidation (June 2018 and November 2019)

The old Taskcluster UI was [updated spawn retrigger action tasks](https://github.com/taskcluster/taskcluster-tools/commit/a6f818087b6030a234c8ab99b5875184fc88d9bd) when `actions.json` was defined. Previously [it showed both](https://bugzilla.mozilla.org/show_bug.cgi?id=1457428), causing confusion. More than a year later, when the new Taskcluster UI replaced the old one on the Taskcluster Community instance, it was [missing the direct retrigger functionality](https://github.com/taskcluster/taskcluster/issues/1892) which [was added back](https://github.com/taskcluster/taskcluster/pull/1893).


## Details

### Implementation Approach

Add `retrigger-method` attribute the full graph is generated. `retrigger-method` can take 2 values: `action-task` or `direct`. 

#### 1. taskgraph/generator.py

Add new attribute [after the full graph is generated](https://github.com/taskcluster/taskgraph/blob/de7ca2bd8a164ec42205964fd3e52863e95c1bb9/src/taskgraph/generator.py#L440-L456). 

#### 2. Taskcluster UI

Modify [`getTaskActionsData()`](https://github.com/taskcluster/taskcluster/blob/c76db520701a19cc49ded480ea3d0cb7378f252b/ui/src/views/Tasks/ViewTask/index.jsx#L151-L182) to keep the old logic AND perform the expected retrigger depending on the value of `retrigger-method`.

#### 3. Treeherder

Same thing with [`JobModel.retrigger()`](https://github.com/mozilla/treeherder/blob/e5a02d39e70ab237bb37c3f497c994b9421f5e83/ui/models/job.js#L89-L163).

### Compatibility

- **Backwards compatible**: Old tasks without `retrigger-method` attribute will continue to old logic

## Open Questions

None.

## Implementation

- [ ] Tracking bug: TBD
- [ ] taskgraph PR: TBD
- [ ] taskcluster PR: TBD
- [ ] treeherder PR: TBD
