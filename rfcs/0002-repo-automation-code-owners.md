# RFC 2 - Repo automation `CODEOWNERS`
* Comments: [#2](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/2)
* Proposed by: @mitchhentges

# Summary

Leverage [Github "code owners"](https://help.github.com/articles/about-code-owners/) for product repositories (e.g.: [`focus-android`](https://github.com/mozilla-mobile/focus-android/)) so that releng is automatically assigned as a reviewer when automation-related files are changed.

## Motivation

Right now, it's possible for product-level changes in automation to happen without releng being notified. This will allow us to give insight to such changes and avoid potential build-breaking accidents.

# Details

We will use the [Github "code owners"](https://help.github.com/articles/about-code-owners/) feature to declare the `automation` (or `tools`) directory to be "owned by" releng. This will assign us as reviewers to changes to this code. 

Example `CODEOWNERS` file:
```
/tools @mozilla-mobile/releng
/automation @mozilla-mobile/releng
/.github @mozilla-mobile/releng
```

This file will be put in `.github/CODEOWNERS`, unless one exists in the repository already.
We will use the mobile-specific @mozilla-mobile/releng team in mobile repositories, and @mozilla-releng/releng as the owner within non-mobile repositories.

We don't want to block merges with our reviews, and we will ensure this on a repository-bases by either:
This will be solved on a repository-basis by either:
* Ensuring that the associated releng team isn't the only owner of the automation files.
* Disabling the option to "Require review from Code Owners" in Github.
![screenshot of github settings](https://help.github.com/assets/images/help/repository/PR-review-required-code-owner.png)

# Open Questions

# Implementation

* [Tracking bug](https://github.com/mozilla-releng/releng-rfcs/issues/15)
