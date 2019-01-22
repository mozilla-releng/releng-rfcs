# RFC 12 - Typing Python Scripts
* Comments: [#12](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/12)
* Proposed by: @mitchhentges

# Summary

[Typing hinting is in the Python standard library as of Python 3.5](https://docs.python.org/3/library/typing.html).
We should organically add type hints to functions and data types that would benefit most, starting within our `*scripts` and `scriptworker`.

## Motivation

Typings have a couple benefits, such as:
* Easing refactoring, since it's easier to see what values are being passed around
* Improved IDE completion

# Details

Just as if introducing Typescript to a Javascript codebase (or Kotlin to Java), we can slowly add typings on a PR basis.
Typed functions interact properly with untyped functions, so refactoring the entire codebase at once isn't necessary.

[PEP 484 has a section on `*args` and `**kwargs`](https://www.python.org/dev/peps/pep-0484/#arbitrary-argument-lists-and-default-argument-values).

Note: typings are ignored in production.

# Open Questions

* Should we using tooling such as `mypy` in CI to statically analyze our typings?
    * Suggestion: perhaps we could add it to CI, but not let it fail the build? Watch this closely, and re-visit whether it's useful in a month or two? 

# Implementation
