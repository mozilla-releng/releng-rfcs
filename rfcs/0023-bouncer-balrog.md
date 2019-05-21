# RFC 23 - Connect the download bouncer to Balrog
* Comments: [#23](https://api.github.com/repos/mozilla-releng/releng-rfcs/issues/23)
* Proposed by: @srfraser

## Summary

We will replace the database and admin page for the bouncer service with Balrog. go-bouncer itself
will query Balrog for data, and cache the results for speedy responses.

## Motivation

1. Bouncer updates would be tied to the release rule signoff in Balrog, meaning both will happen simultaneously.
2. bounceradmin is aging and has operational issues.
    1. There is no sign-off, so a single actor controls bouncer behaviour.
    2. SOCKs proxy for access
    3. Old codebase with support issues
    4. Database management is required
3. We could remove the bouncer submission, location and aliases tasks from the release graphs.


## Details

The first tool to modify is [go-bouncer](https://github.com/mozilla-services/go-bouncer).
Deployment and management of the go-bouncer service does not change, but it will now contact Balrog
instead of MySQL.

go-bouncer will do the following:

1. Receive a query with the current URL format (example below)
2. Extract the query string to determine product, OS and locale.
3. Look up the product and OS in its local cache, this will find a template URL to complete and an appVersion
4. Use the template, appVersion, a product name map, the locale and OS to fill in the template.
5. 302 to the filled in template URL, or error if it is not available.

For example, if bouncer gets a query for `https://download.mozilla.org/?product=firefox-latest-ssl&os=osx&lang=en-US`
it will find:

1. A query product of `firefox-latest-ssl` which the config says should be rewritten as `firefox` in the template.
2. A query OS of `osx` which along with the product lets us look up the right template to use
3. A `lang` which we can validate, if needed.
4. The fields above let us find the latest version in the cache, and a template, such as `/pub/{product}/releases/{version}/mac/{lang}/Firefox {version}.dmg`.
5. The bouncer environment contains the base CDN url, such as `https://download-installer.cdn.mozilla.net`
6. We now have all the details to fill in the template.

At regular intervals and at startup, go-bouncer will query Balrog to discover up-to-date information.

For configuration that doesn't depend on Balrog, we will use a static configuration file. How this is deployed will is to be decided.
Here is an _incomplete_ example of what a configuration file might look like.

```
{
    "update_interval": 60,
    "balrog_url": "https://aus5.mozilla.org/api/v1/",
    "product_to_rules": {
        "firefox-latest-ssl": "firefox-release",
        "firefox-devedition-latest-ssl": "devedition",
        "firefox-beta-latest-ssl": "firefox-beta",
        "firefox-nightly-latest-l10n-ssl": "firefox-nightly"
    },
    "product_names" = {
        "firefox-latest-ssl": "firefox",
        "firefox-devedition-latest-ssl": "devedition",
        "firefox-beta-latest-ssl": "firefox",
        "firefox-nightly-latest-l10n-ssl": "firefox"
    },
    "templates": {
        "default": {
            "osx": "/pub/{product}/releases/{version}/mac/{lang}/Firefox {version}.dmg",
            "win64": "/pub/{product}/releases/{version}/win64/{lang}/Firefox Setup {version}.exe"
        }
        "firefox-nightly-latest-l10n-ssl": {
            "win64": "/pub/{product}/nightly/latest-mozilla-central/firefox-{version}.{lang}.win64.installer.exe
        }

    }
}
```

Every `update_interval` seconds, or at startup, refresh the cache:

```
For every entry in the `product_to_rules` map
    Use the value as a rule alias to query {BALROG_URL}/rules/{rule_alias}
    The rule contains the currently active release name
    Fetch the data for {BALROG_URL}/releases/{release_name}
    Store appVersion and supported locales in the cache for current product value.
```


## To Deploy

1. Deploy a staging version of the new go-bouncer
2. Run a modified version of the check-bouncer script against it to ensure all the results are valid and the same as the live version for a variety of products
3. Deploy to the live environment.
4. Remove the unnecessary tasks from all graphs
5. Decommission bounceradmin and the backend database.


### Nightlies

Bouncer does not currently use any of the dated directories for nightlies, meaning this complicated option is not required. The 'latest' directory is sufficient and can be used in the templates.

### Bouncer URLs for specific versions

Bouncer handles redirection for older versions, by including this as a product in the URL. For example, a query to `http://download.mozilla.org/?product=firefox-65.0-complete&os=%OS_BOUNCER%&lang=%LOCALE%`

Since there are no Balrog rules pointing to these versions, and Balrog itself relies on bouncer to find the locations, we need to convert these into templates.

Thankfully, having a product of the form `firefox-65.0-complete` means we can easily extract the version instead of querying Balrog for it. We wouldn't be able to validate
the list of locales, but we could use the same templates to fill in the locations. This does assume that the URLs follow a consistent pattern. If older ones do not, we may have to hard code them.

Partials in products would also need to be accounted for, to translate queries such as `http://download.mozilla.org/?product=firefox-65.0-partial-63.0.1&os=win32&lang=en-US` into `http://download.cdn.mozilla.net/pub/firefox/releases/65.0/update/win32/en-US/firefox-63.0.1-65.0.partial.mar`

This means that when looking up a product in the local cache, we first check to see if there's an exact match, and then check to see if there's a pattern match against a rule name.

## Open Questions

1. How do we detect an invalid cache?
2. Should the cache be in-memory only?
3. Should we be caching the valid locales, for validation, or just allow a 404 at the destination?
4. What should the behaviour be for invalid product/locale/platform combinations?
5. How should we deploy the configuration file?
6. Is it a safe assumption that the url templates will not change frequently? If they will, a static configuration file may not be a good idea.
7. Should we support the dated nightly directories? This may have helped us during the Armag-addon weekend.
8. Should this change also include instrumentation, such as logging successful queries, and the platform/OS/locale combinations requested?

## Implementation

<once the RFC is decided, these links will provide readers a way to track the
implementation through to completion>

* <link to tracker bug, issue, etc.>
* <...>

