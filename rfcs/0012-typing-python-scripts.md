# RFC 12 - Typing Python Scripts
* Comments: [#12](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/12)
* Proposed by: @mitchhentges

# Summary

[Typing hinting is in the Python standard library as of Python 3.5](https://docs.python.org/3/library/typing.html).
We should allow type hints to be allowed organically to functions and data types. Additionally, `mypy` will be set up
within `scriptworker` and its `*scripts`.
Note that adding type hints for all existing code (or enforcing it for new code) is **not** part of this RFC. Instead,
this is purely meant to allow type hints to be used and supported where they're most valuable, but code that doesn't
benefit from hinting can be left un-hinted.

## Motivation

Typings have a couple benefits, such as:
* Easing refactoring, since it's easier to see what values are being passed around
* Improved IDE completion
* Fragile code can be significantly more stable when supported by type hints

# Details

This RFC does not recommend fully type-hinting our codebases. This is purely about supporting piece-by-piece additions
of type hints.

`mypy` will be added to `scriptworker` and the `*scripts` and will be invoked on-PR, and will block merges if the
static analysis fails.

If a PR adds typing that triggers faults in `mypy`, we can simply remove those type hints until the tooling improves.

# Open Questions 

# Implementation

* [Tracking bug](https://github.com/mozilla-releng/releng-rfcs/issues/28)
