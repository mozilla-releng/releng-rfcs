# RFC 13 - Proposal to Disable Fennec Balrog Submission
* Comments: [#13](https://github.com/mozilla-releng/releng-rfcs/issues/13)
* Proposed by: @Callek

# Summary

I am proposing that we disable Fennec Balrog submission, which causes there to be no offered updates for new
Fennec builds when Fennec has been sideload installed.

With this work we will do all of the following:

* Disable Updates on **Single Locale** Fennec builds on `nightly` channel
* Disable Updates on **All** Fennec builds on `nightly` channel
* Disable Updates on **All** Fennec builds on `beta` channel
* Disable Updates on **All** Fennec builds on `release` channel

## Motivation

There are a handful of configuration differences and bugs that make Sideload Install updates difficult to manage, both
from a product perspective and from the Release Engineering side. Explicitly no longer supporting automatic updates
via Sideload install will simplify this work.

I came across a few bugs resulting in single locale and non-en-US multi locale updates being busted on Fennec
Nightly via Balrog right now. This has seemingly gone unnoticed so user impact appears to be low. More
details on those bugs and current user metrics are highlighted below:

* (***non**-en-US/multi*) updates via the channel `nightly-mozilla-central` which Balrog does not currently recognize,
  would need to change to `nightly` for product and rescue the users in balrog
     * <s>Fix ETA < 2 days of work.</s>
     * Fixed in [Bug 1520874](https://bugzil.la/1520874) ([changeset](https://hg.mozilla.org/mozilla-central/rev/3c3a5c19f715))
* (***non**-en-US/multi*) `nightly` update submissions via balrog are disabled due to the "Declarative Artifact" landing since 11-28.
     * Bug to be filed if balrog is kept for these, effort to fix ~ a week.
* (*en-US and multi*) while `en-US` is produced as a stand-alone *single locale* build, it shares an update url
  with the *multi-locale* build, meaning that any user who installs `en-US` will get a balrog update to *multi-locale* anyway.
    * ETA unknown, fix may involve product changes to use the selected language, or may be a change to what we query
      balrog with for multi.
* The updater is explicitly disabled in product for Fennec in official builds for `beta` and `release`.
    * See [Bug 1520248](https://bugzil.la/1520248)

# Details

When looking at actual data (balrog request counts) ...

Articulating per-channel update requests [since Jan 1’st 2019 until Jan 21'st - 3 weeks] (logged whenever
a background update check is initiated for fennec, or when a user clicks 'check for updates' in UX).
(Data was pulled from balrog access logs, see raw data: [total channel counts](https://sql.telemetry.mozilla.org/queries/61044/)
and [Per-Version counts of different channels](https://sql.telemetry.mozilla.org/queries/59001/))

This is **NOT** user counts but request counts, unique users with sideload installs is an unknown smaller % of these numbers.

These numbers also greatly reduce in quantity when including only recent versions.   (this data expressed inside the `{}` below)

* `nightly`: **171,250** requests  {**99,175** requests on *66.0a1*, **23,230** requests on *65.0a1*}
* `nightly-mozilla-central`: (*single-locale*) **843** requests  {**355** requests on *66.0a1*,  **142** requests on *65.0a1*}
* `aurora`: **17,272** requests  {**0** current version requests}
    * `aurora` users specifically are latest-version of *54.0a2*, so these users are **all** older than Jun 2017.
* `beta`: **63,253** requests  {**2** requests on *65.0*, **2** requests on *64.0*}
* `release`: **3,053,198** requests   {**12** requests on *64.0.x*, **0** requests on *63.0.x*}

# Open Questions

...?

# Implementation

Unless no new information as to reasons we shouldn't do this, I will be going ahead with implementation with intended landing
on Feb 5, 2019.

* Disable Balrog Submission of new builds for all channels
   * Bug XXXX
* Disable in-product updater for all version of Fennec
   * Bug XXXX

* <link to tracker bug, issue, etc.>
* <...>

# FAQ

* I'm on a now-stranded build of Fennec, what do I do?
    * Automated updates will only be supported via your devices app-store (Such as Google Play) if your device does
      not have a supported App Store you will have to manually install new versions as they are available.
    * To Migrate from a sideload install to an app store install:
        1. Setup Firefox Sync
        1. Install from your App Store (Google Play)
        1. Setup Firefox Sync again
        1. Sync your data back.
