# RFC 32 - taskgraph: Detect when builds from a previous push can be reused
* Comments: [#32](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/32)
* Proposed by: @JohanLorenzo

# Summary

Taskgraph, in the decision task should be able to retrieve a previous build if the changeset is a test-only one.

## Motivation

Per [1]:
> Approximately 13-14% of pushes to the try and autoland branches are test-only pushes. If test repackaging can be done without running builds, this can reduce the CI costs.

# Details

## In a few words

The decision task looks at the push it's dealing with and gets:
 * the revision before this push
 * the files that were changed by this push.

If none of the changed files is a source file, then the decision task will get the reuse the builds made in the previous push.


## Workflows

### Happy path

Conditions:

 a. The previous push produced builds.
 b. All builds from the previous push are completed.
 c. No platform is missing in the previous push.
 d. The previous decision task has already run.
 e. The previous decision task is green.
 f. We are confident the previous builds are good.


The decision task runs. Everything goes as usual up until the optimization phase. During this one, taskgraph gets the push metadata thanks to the `json-pushes` endpoint[2], and gets the list of changed files. If a source file is present, then the build tasks in the graph remain unchanged. Otherwise, it gets the hash of the parent of the last changeset (no need to make a new request, it's available in the previous response). Then, taskgraph talks to the Taskcluster index to get the corresponding builds. The indexes will contain (if they don't already) the revision so that's how taskgraph will easily look them up. Then, taskgraph optimizes away the build tasks, just like we do with Docker images. Finally, it injects some `insert-indexes` tasks to point this revision to the builds it uses. This will be important for the next case.


### The previous push also re-used builds.

Condition a. is changed. A new push comes in. Taskgraph processes it the same way as described above: it gets the hash of the parent of the last change fo the current push. It finds existing builds on the Index thanks to this very hash, because it was populated by the `insert-indexes` task. Thanks to this logic, condition a. is not important anymore, it's easy to deal with both cases. Let's ignore it from now on.


### Builds of the previous push aren't completed yet

Condition b. is changed. A new push comes in. By the time the decision task runs, builds are not done yet (they may have even failed). In the current way we use indexes, we have no way to tell that a given route is being populated by another task. That's something Callek calls "eager indexes" and he filed bug 1653058, for it. Thus, this current bug depends on it. Once this is solved, taskgraph will just have to listen to the right indexes to be sure to wait on upcoming builds.


### The previous push didn't build all platforms

Condition c. is changed. We don't care about condition a. and b., here. We may end up in situations where just a single platform was built in the previous push. In this case, the previous push will add `insert-indexes` tasks for any optimized-away platform.


### The previous decision task is still running

Condition d. is changed. We may end up in situations where the current decision tasks starts while the previous is still running. That can easily happen on try or on autoland. We cannot use the index of the previous decision task because it's yet to be populated. This means, for any missing previous build, we need to make sure that the decision task was completed in the first place. Then, if it's not, we need to make sure it's not running. We could use Treeherder to know piece of data, but that ties taskgraph to another service. We could also change `.taskcluster.yml` to spin up a dummy task along the decision one to "eager index" it. I kind of like the latter, but I wonder if Chain of Trust may be broken because of it.

### The previous decision task has failed and a rerun won't fix it

Condition e. is changed. The previous push may just turn red and we can only salvage it with another push. The "eager index" created in the previous paragraph tells taskgraph the previous task has started. Sadly, since the decision task remains red, there is no way to populate the index with that broken task. So taskgraph needs to know when to stop waiting for the previous decision task. A simple solution could be to bake the `taskId` of the previous decision task in the payload of the previous dummy task. It's quite easy to do with `.taskcluster.yml`. This way, taskgraph know where the dummy task is, thanks to "eager indexes", then it knows the `taskId` of the previous decision task so it can monitor it.


### Sheriffs want to respin builds on a test-only changeset.

Condition f. is changed. For whatever reasons, we want to make builds on a revision that got its builds optimized away. This is going to be handled in an action task. I don't know whether `rebuild-kinds` will work out of the box, or if we need to pass a custom parameter to tell taskgraph to not optimize away them once more. In any case, it should be a matter of guarding the optimization code with an `if` statement.


### Any other edge cases?

I currently don't see any edge case that isn't covered in the setup hg + hg.mozilla.org. Feel free to tell me if you see something missing :)



## Local runs

Callek pointed out something really important when we chatted: people must be able to run taskgraph while being offline. So any calls made to hg.mozilla.org must be translated into some hg commands that run locally. I know `hg wip` provides something close to `json-pushes` in the sense that you know what your current head was based off of. So, we can get the last public changeset and taskgraph will point optimized-away builds to this repo/revision. I just need to figure a more machine-friendly command. If any of you know what this hg command is, I'll be happy to hear from you :)

I see one issue with that: we won't be able to know what revisions were already pushed to try. In fact, pushing there doesn't make any changeset public (which is good). So the result you have locally, may not be the same as the one the decision task created. The decision will be able to optimize 2 try pushes away thanks to the `json-pushes` endpoint.



## Translating the proposal to git

Rail made a point when we chatted: even though we want this to work on hg (and more precisely hg.mozilla.org), we still want to support git (and github.com) workflows for projects like Fenix. Good news is: GitHub now exposes pushes metadata thanks to this new API[3]. So we should be able to implement a similar logic than the one with `json-pushes`. The main issue, though: we need to give a GitHub token to the decision task. The number of unauthenticated requests is way too low to the scale of Taskcluster. Authenticated requests may be not enough either, because one user has a certain amount of requests he/she/they can make. So may need several bot accounts to absorb the full load.




# Open Questions

 * Will Chain of Trust break if `.taskcluster.yml` spins 2 tasks instead of a single one?
 * Does `rebuild-kind` get optimized away?
 * Any other edge cases?

# Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>




[1] https://docs.google.com/document/d/1dVhDPCe3ibEJzD9VZ47VUMCy4s18NbUgeLnYJFNQtTU/edit
[2] E.g.: https://hg.mozilla.org/mozilla-central/json-pushes?version=2&changeset=56082fc4acfacba40993e47ef8302993c59e264e&full=1
[3] https://developer.github.com/v4/object/push/
[4] https://developer.github.com/v4/guides/resource-limitations/#rate-limit
