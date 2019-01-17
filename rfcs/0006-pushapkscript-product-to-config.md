# RFC 6 - Pushapkscript move product-specific behaviour to config
* Comments: [#6](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/6)
* Original RFC: [in `pushapkscript`](https://github.com/mozilla-releng/pushapkscript/issues/65)
* Proposed by: @mitchhentges

# Summary

`pushapkscript` and `mozapkpublisher` should behave as tools that push/publish _apks_. I think it's great that they have different checks and verifications, but I think that these should be provided in `pushapkscript` config and as runtime options respectively, rather than being hard-coded in the source code.

This means that we'll need to:

* Add additional options to `mozapkpublisher`
* Update users of `mozapkpublisher` to use these new options
* Expand options in `pushapkscript` config, map relevant options to new `mozapkpublisher` options
* Update `pushapkscript` config in `build-puppet`

## Motivation

We have a few places [in `pushapkscript`](https://github.com/mozilla-releng/pushapkscript/blob/master/pushapkscript/script.py#L34-L41) and [in `mozapkpublisher`](https://github.com/mozilla-releng/mozapkpublisher/blob/master/mozapkpublisher/common/apk/checker.py#L19-L57) where we change the behaviour of the script based on the product. This has a couple downsides:

* When adding/changing a product's checks, new versions of `mozapkpublisher` and `pushapkscript` need to be released
* Configuration per-product is scattered across many repositories: the main repo (e.g. [`reference-browser`](https://github.com/mozilla-mobile/reference-browser)), [`build-puppet`](https://github.com/mozilla-releng/build-puppet/), [`pushapkscript`](https://github.com/mozilla-releng/pushapkscript/), and [`mozapkpublisher`](https://github.com/mozilla-releng/mozapkpublisher/)
    * Adding new products requires a lot of changes to many different places - unless you know where to look, it's hard to track 'em all down
    * New releng members (e.g.: "Budget Mitch/Budget Budget Jlund" :P) have an increased barrier of knowledge required to understand how projects are built/deployed, and where to contribute different changes

# Details

**Technical Details - hard-coded product usages**

* [`pushapkscript` - `_AUTHORIZED_PRODUCTS_TO_REACH_GOOGLE_PLAY`](https://github.com/mozilla-releng/pushapkscript/blob/5b2258d529caf79b49aa4014fd77d1b39db9a571/pushapkscript/googleplay.py#L10)

I'm not sure what value this configuration adds - if a product isn't allowed to reach google play, then it won't have a `service_account` and `certificate` in the `pushapkscript` config. Perhaps this can just be removed?

* [`pushapkscript` - `_DIGEST_ALGORITHM_PER_ANDROID_PRODUCT`](https://github.com/mozilla-releng/pushapkscript/blob/5b2258d529caf79b49aa4014fd77d1b39db9a571/pushapkscript/manifest.py#L21)

We could add a `digestAlgorithm` property to the `config` for each product

* [`pushapkscript` - uploading strings](https://github.com/mozilla-releng/pushapkscript/blob/5b2258d529caf79b49aa4014fd77d1b39db9a571/pushapkscript/script.py#L34-L41)

We can add a boolean property like `includesGooglePlayStrings` to `config` for each product

* [`mozapkpublisher` - cross checks](https://github.com/mozilla-releng/mozapkpublisher/blob/master/mozapkpublisher/common/apk/checker.py#L24)

Provide a list of `skipChecks` in the `config` for each product. 

* [`mozapkpublisher` - `extract_metadata`](https://github.com/mozilla-releng/mozapkpublisher/blob/master/mozapkpublisher/common/apk/extractor.py#L44-L53)

Not sure how this can be handled elegantly - perhaps inferred from `skipChecks`? Not sure how all the properties are used. Worst-comes-to-worst, just have a property in the `payload` listing which properties we care about extracting, then refactor this further/properly in in the future

* [`mozapkpublisher` - `_ADDITIONAL_TRACK_VALUES`](https://github.com/mozilla-releng/mozapkpublisher/blob/master/mozapkpublisher/common/googleplay.py)

We can provide this in `config` for each product, though then we'll just be providing both `track` and `setOfValidTracksToAssertAgainst`, which is kind of like:

```
let track = 'nightly'
let valid_tracks = ['nightly']
assert track in valid_tracks
```

So perhaps this can be removed? Worst-comes-to-worst, add `additionalTracks` to `payload`, refactor in the future

# Open Questions

# Implementation

* [Tracking bug](https://github.com/mozilla-releng/releng-rfcs/issues/9)



