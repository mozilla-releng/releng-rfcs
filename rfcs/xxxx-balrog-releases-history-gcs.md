# RFC <number> - Move Balrog's releases_history table to Google Cloud Storage
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @mozbhearsum

# Summary

Move Balrog's `releases_history` table from its current location as a table in the MySQL database, and into Google Cloud Storage (GCS).

## Motivation

`releases_history` is an extremely large, infrequently used table (~450GB, more than 99% of Balrog's database). Migrating it to GCS has a number of benefits:
* Ease our migration of Balrog into Google Cloud Provider (GCP), because the database will become small enough that downtime or disabling writes will be a non-issue.
* Simplify retention with GCS Lifecycle Policies. These would fully replace our Python code that currently purges data from `releases_history`.
* Speed up access to Release History, because GCS access is faster than going through the admin backend and database. (see open question about this)
* Allow for local Balrog installations to access full Release history.

# Details

## Architecture

The simplest way to understand the proposed changes is to look at the flow of data now, and how it would look with the proposed changes. In the current world, everything that reads or writes Releases history goes through the admin backend to do so, as shown here:
```
+---------------+     +---------------+     +------------+
|               |     |               |     |            |
|   Frontend    |     |    Balrog     |     | Public API |
|               |     | Scriptworkers |     |            |
+------+---+----+     |               |     +------+-----+
       |   ^          +-------+-------+            ^
       |   |                  |                    |
       |   |                  |                    |
       |   | Reads Release    | Updates Release    |
       |   | History          |                    |
       |   |                  v                    |
       |   |              +---+-----+              |
       |   +--------------+         |              |
       |                  |  Admin  +--------------+
       +----------------->+         | Reads Release History
        Updates Release   +--+---+--+
                             |   ^
                             |   | Reads & Writes Release History
                             |   | to releases_history Table
                             v   |
                          +--+---+--+
                          |         |
                          |  MySQL  |
                          |         |
                          +---------+
```

With the proposed changes, only writes go through the admin backend, and are ultimately handled by the Agent. Anything wishing to do reads, does so directly with GCS:
```
+---------------+
|               |
|    Balrog     +----+                         Reads and Writes
| Scriptworkers |    | Updates                 Release History to
|               |    | Releases    +---------+ a Temporary Table +---------+
+---------------+    +------------>+         +------------------>+         |
                     +------------>+  Admin  |                   |  MySQL  |
+---------------+    |             |         +<------------------+         |
|               |    |             +----+----+                   +---------+
|   Frontend    +----+                  |
|               |                       |
+-------+-------+                       | Reads Release History
        ^                               | from Temporary Table
        | Reads Release History         |
        |                               v
   +----+----+                     +----+----+
   |         |                     |         |
   |   GCS   +<--------------------+  Agent  |
   |         | Writes Release      |         |
   +---------+ History             +---------+

```

## GCS

Buckets will be configured as follows:
* One bucket for Nightly Releases, with a 14 day Lifecyle Policy for all keys
* One bucket for all other Releases, with no Lifecycle Policy (ie: they will be kept indefinitely)
* Both buckets will be made publicly available, as Releases contain no secrets
* We will create a folder at the top level of the appropriate bucket for each Release. Eg: Firefox-66.0.2-build1
* Within each folder, there will be a key per data version with basic data (data version, timestamp, changed_by) encoded in the name
    * An example of a full key name (with folder) is: Firefox-66.0.2-build1/547-1553640087550-balrog-ffxbld.json
* Key values will be the Release `data`.

The dev and stage enviroments will have their own sets of buckets. It's important that these environments are able to fully simulate production, so sharing the production buckets is inappropriate here. However, we don't need to care about long term retention of any of their data, so they will be configured to retain it for only 5 days (including for non-nightlies).

Local development environments will *not* have their own buckets by default (more on that in the Caveats section).

## Balrog Changes Needed

* Replace writes to `releases_history` table with writes to a temporary table.
* Enchance the Agent to monitor for these rows and write them to GCS.
* Numerous changes to the Frontend:
    * Query GCS when Release history is requested instead of admin. The appropriate bucket can be queried by prefix to find all versions of the Release.
    * Change the way we do reverts. Instead of calling a special admin endpoint that copies data from `releases_history` to `releases`, we'll simply have the Frontend update a Release with the old data (or schedule a change).
    * Generate diffs itself, rather than asking the admin to do it.
    * Stop showing Product in the history UI (it cannot be changed, and removing it avoids the need to encode it in the key names).
* Do a one-off import of existing non-Nightly history to GCS.
    * Because Nightly history is purged so quickly it's probably not worthwhile to import it.

## Caveats
* Local Development environments will use the production Release history. This means that by default, they will _not_ be able to write new Release history.
    * Given how infrequently we make changes that code, this should be OK.
    * If/when we do need to test writing Release history locally, it will be possible to configure it to do so.
* The Public API (https://aus-api.mozilla.org) will no longer provide Releases history. Consumers of it must learn how to look for it on GCS if needed.
    * There are no known consumers of Releases history through this API, so this change shouldn't cause any disruption.
    * This limitation means that web cluster does not need to know anything about GCS.

# Open Questions

* Will access to Release History actually be faster? Need to compare requests to the admin app vs. requests to GCS to verify.

# Implementation

TODO
