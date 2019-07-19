# RFC <number> - Class Objects as Arguments
* Comments: [#<number>](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/<number>)
* Proposed by: @mitchhentges

# Summary

Johan and I are having an architecture discussion around [this PR](https://github.com/mozilla-releng/mozapkpublisher/pull/215).
Specifically, there is a public function ([`push_apk(...)`](https://github.com/mozilla-releng/mozapkpublisher/blob/f41ad26b508395c88e222d4f231fe31e2abadcef/mozapkpublisher/push_apk.py#L13-L26)) 
on a library (`mozapkpublisher`) that's used by both a command-line interface and a Taskcluster worker interface.

Importantly, this function has a significant amount of parameters which have different constraints depending on the
value of _other_ parameters. For example, `service_account` and `google_play_credentials_file` should not be set if
`contact_google_play` is `False`, but are **required** if `contact_google_play` is `True`.

For example:

```
push_apk(                                           |       push_apk(
    contact_google_play=False,                      |           contact_google_play=True,
    service_account='me@google.com',               O R          apks=[...],
    google_play_credentials_file='creds.p12',       |           ...
    apks=[...],                                     |       )
)                                                   |       
```

For consumers of this function, the nuances aren't easily understood and they either have to lean on documentation or
read the function source code.

Instead, I'm advocating for using the type system to be more explicit about expectations:

```
push_apk(                                           |       push_apk(
    GooglePlayConnection(                           |           DryRunConnection(),
        'me@google.com',                           O R          apks=[...],
        'creds.p12',                                |           ...
    ),                                              |       )
    apks=[...],                                     |       
    ...                                             |
)                                                   |
```

## Motivation

(note: when I mention "object", I'm referring to class objects)

**Benefits of using classes to represent intent:**
* Expected parameters are more explicit and self-documenting - you don't need to depend on docs to tell you that 
`credentials` is only needed when `contact_google_play=True`
* Parameters that are related (like `credentials` and `service_account`) are grouped [note 1](#note-1-using-a-dictionary-instead-of-an-object-as-a-parameter)). This is especially
useful when expressing intent, as in "I want to do `<x>`, and here's the associated configuration for doing `<x>`."
* IDEs can use the class definition to provide Intellisense

When discussing with Johan, he mentioned some potential drawbacks when using classes:

**Downsides of using classes:**
* Functions that take data class objects as parameters are more annoying to test, since you need to provide all the
parameters. If a `dict` was used instead of a class, unnecessary properties could be omitted for the tests
    * FWIW, if you're using classes, you have the option of filling in unused parameters with `None`. `dict`s can do
    this with less code, since any options you don't explicitly specify are automatically `None`. However, by achieving
    this benefit, you'll incur all the costs of [using a `dict` in your arguments](#note-1-using-a-dictionary-instead-of-an-object-as-a-parameter)
* When requiring that consumers provide objects, you're forcing the data to be minimally validated and massaged 
before your function is called. This means that each module using your function has to write similar logic to create 
the objects. If primitive parameters were used, the validation could be re-used within the function.
    * I'm not sure that there's a lot of benefit in re-using that minor validation and massaging code, since it forces
    the calling interfaces to be coupled. For example, though the `contact_google_play` variable may make sense in
    config for a Taskcluster worker, you might have the CLI automatically do a dry run if credentials aren't provided.
    By splitting this validation, you're allowing your two interfaces to evolve separately to meet their individual
    requirements
        * Note: I'm not advocating against sharing any validation. I like how `mozapkpublisher` handles APK validation,
        regardless of what interface is requesting that an APK is pushed to Google Play. However, I think validating
        that the correct arguments are set based on the value of other parameters (which is information that types can 
        represent) is validation that isn't valuable being shared as function code.
* If objects are used, they need to be imported before they can be created. Some code editors can prompt to
automatically add a missing import, but if your editor doesn't have this functionality, objects imports will need to be
manually added. This could add significant overhead to productivity.
    * If a language feature can improve maintainability, I strongly feel that we should use developer tools that don't
    restrict us from embracing such a language feature.

### Note 1: using a dictionary instead of an object as a parameter

Most of the main [motivation](#motivation) discussion is comparing leaving related values as separate parameters vs
grouping them in a class. This section considers the third option of using a `dict`.

Representing a data structure as a `dict` is similar to using a class, since you're able to group parameters into
their logical structure. However, you run into some downsides:

* Most importantly, a `dict` isn't explicit. It's not clear what properties it's expecting, or when they're optional.
    * When a consumer needs to provide such a `dict`, they're forced to rely on good documentation, or they need to
    hunt through source code to make sure that they're providing a `dict` with the appropriate structure
* When using values from a `dict`, they could be in one of three states: The value is defined, the value is `None`, or
the value doesn't exist on the `dict` at all. This adds a tertiary state that you have to handle
    * Johan mentioned that this could be sidestepped by doing `group.get('prop', None)`, but this adds a lot of noise to
    each property access compared to `group.prop`.

When handling dynamic data, or when doing some functional-style programming, a `dict` may make sense. However, in this
case, where we're dealing with a function on an API boundary, I feel that being explicit is more beneficial.

# Details

No implementation, this is an architecture discussion that affects future work, but isn't about any particular goal

# Open Questions

* Do we want to use classes and the type system to represent intent as advocated in this RFC, or would we rather
lean on language primitives?

# Implementation

_None_

