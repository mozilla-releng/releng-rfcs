# RFC <number> - Typing Python Scripts
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @mitchhentges

# Summary

[Typing hinting is in the Python standard library as of Python 3.5](https://docs.python.org/3/library/typing.html).
We should slowly introduce it into our `*scripts` and `scriptworker`

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

# Implementation
