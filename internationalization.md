# Internationalization (i18n)

- Use UTF-8 everywhere.

- Determining language:
  - Geolocation? browser language? just ask? detecting a user’s locale is not always the best option.
  - Provide users with an option to switch between languages easily.

- Translation dictionaries
  - Repo and deployments: English (default) localization file should be in the repo, but the rest should be deployed separately (eg. straight to CDN).
    - Default language is required, as a broken default lang breaks the site, the tests, etc.
    - But for the rest:
      - Translations are updated by a different team, not devs.
      - No need to wait for the sprint release. Issues in the files would bring the default lang copy.
        - Fallbacks can be tracked and alerts can be set.
      - Translations should not block dev deploys. Some languages can take time and eg. you only added a new validation or error copy.
  - Format: Should be compatible with translation services we might use in the future, eg. Phrase, Transifex, etc.
    - JSON, YAML, etc. are good options, but check what services you'd like are using.
    - Beware gettext, it's a bit of a pain.
  - Reuse keys?
    - Reusing keys like "yes", "no", "next", "previous", "ok" and "cancel" may be ok but beware reusing more specific keys, as changing them may affect unexpected places.
  - Nesting keys?
      - It may make sense to nest by component, but think the structure through at the beginning: if you also nest by page, you have a conflict when you use the same component in different pages. I'd go `business-domain > component > internal-component-structure...`.
        - Using yaml, you can use `&anchors` to `*refer` to definitions made elsewhere, but again beware surprises

- UI Layouts
  - Sizing
    - Copy length can differ (seen 0.5x to 2-3x English length quoted somewhere).
    - Required font size may be different (eg. kanji).
    - Will you support right-to-left langs? see below about the body css class.
    - A typical solution is pseudo-localization: have a "test language" using UTF characters like "!!! Àççôûñţ Šéţţîñĝš !!!" for "Account Settings". See more at [Wikipedia](https://en.wikipedia.org/wiki/Pseudolocalization).
  - Text in images is tricky:
    - CSS text overlays are possible but again beware different text sizes, etc.
    - Localized `hrefs` pointing to a bunch of different images may be a nicer option allowing freedom to the designer, but actually relies on designers doing all of them (process issue: workload, time, etc.).
  - Colors are locale-specific also. Red and green don't mean the same thing everywhere: while the color red generally means love, compassion, enthusiasm, excitement, and danger in European countries, it means prosperity, wealth, and success in China.
  - A typical solution is to add the locale name as a class to the body, so it's easy to code particularities in CSS.

- Dynamic translations
  - Watch out for variable substitution. Don't split your strings. Leave them whole like this: `"You have %{count} new messages"` and replace the `%{count}` with the number.
  - Different languages have different pluralization eg. 0: zero, 1: one, 2-4: few, 5-7: many, 7-: other. See the [plural rules at unicode.org](https://www.unicode.org/cldr/charts/45/supplemental/language_plural_rules.html).
    - Have a tool that allows handling those rules, yielding "one", "few", "many" etc. for numbers, and use it _only_ to retrieve the corresponding key. See the [example below](#i18n-example-showing-foreign-pluralization-and-other-localization-formatting).
  - Ordinals are similar.
  - Dates, times, numbers, currency and units (before or after, etc.) are normally handled by the language or other localization tools. Use them too.
    - Consider adding things like ordinals or date formats to the language files.

- Obtaining translations
  - Context is difficult. Sometimes localizers need to know where/how a string is used to make sure it's translated correctly.
    - Services exist for this, allowing translators to work on top of the actual website with a js overlay, but may be expensive for an initial stage
      - A simple solution is to have a 'context' or 'comment' field in the translation file
      - The dict format we pick should support this in the future (see above)
  - If not using a service like Phrase or Transifex, and if we opt for something like JSON for lang files, translators may need to use a tool to edit those files. Talk to them. Tools exist.

- Pulling translations from the backend to the frontend
  - We may pull all keys at once, though that can be too big
    - We may split by page or component but if we reuse keys, key selection can be cumbersome
      - By component sounds nice in my head now, but beware notes above regarding key reuse and nesting!
  - We may use an API, but we'd need a way to bundle requests for many keys

- SEO
  - Search engines should be capable of indexing views rendered in JS nowadays but care should be taken for public routes. It _may_ be justifyable to:
    - Use server-side rendering, or
    - Render the first view in the default language and then switch to the user's language
  - We may need to translate URLs, meta tags, etc.
    - Depending on what's needed, this may mean the i18n tooling should be integrated to the router for dynamically generating routes.
        - Complex: many implications if following that path: redirects when changing routes? etc.

- Testing
  - Automated tests for translations are possible, but not easy.
    - You can check for missing translations, but not for bad ones.
    - You can check for missing pluralization keys, variables, etc. but not for wrong ones.
    - You get the idea...
  - May also not be super valuable. If you have a good process, you can fix them quickly.

- Errors
  - Errors in translations can offend people, break the UI, etc.
    - Mid-long term you need a process for fixing them quickly.
      - A 'comment' field in the translation file can be used to collect the feedback and warn about possible bad translations.

## I18n example showing foreign pluralization and other localization formatting

This is no particular tool, tough would be easy to implement. The idea is to exemplify the required functionality and typical usage.

```yaml
# en-US.yml
events:
  notification:
    zero: "You have no upcoming events"
    one: "You have one upcoming event on %{date}"
    other: "You have %{count} upcoming events, the next one is on %{date}"
payments:
  modal:
    step3:
      confirm-btn: "Confirm"  
```

```yaml
# ru-RU.yml
events:
  notification:
    zero: "У вас нет предстоящих событий"
    one: "У вас одно предстоящее событие %{date}"
    few: "У вас %{count} предстоящих события, следующее %{date}"
    many: "У вас %{count} предстоящих событий, следующее %{date}"
    other: "У вас %{count} предстоящих событий, следующее %{date}"
payments:
  modal:
    step3:
      confirm-btn: "подтвердить"
```

```js
// Somewhere in the initialization of the JS bundle
// locale = "ru-RU"
translationsHash = await fetchLanguageFile(locale)
i18n.init(locale, translationsHash);
```

Contrived example first, to show how complex it can be:
```js
// events-notification.js (a component)

// eventCount = 5
// nextEventDate = new Date(2022, 11, 31)
const pluralizationKey = i18n.pluralizationKey(eventCount);                                                  // "many"
const notificationTemplate = i18n.translate(`events.notification.${pluralizationKey}`);                      // "У вас %{count} предстоящих событий, следующее %{date}"
const dateOptions = { year: "numeric", month: "long", day: "numeric" };
const localizedDate = nextEventDate.toLocaleDateString(locale, dateOptions);                                 // "31 декабря 2022 г."
const notification = notificationTemplate.replace("%{count}", eventCount).replace("%{date}", localizedDate); // "У вас 5 предстоящих событий, следующее 31 декабря 2022 г."
```

Of course that you can make this less verbose aliasing i18n methods and inlining. Simple translations should be easy:
```js
const t = i18n.translate;                         // And other aliases, somewhere in the initialization of the JS bundle
const i18n.setKeyPrefix("payments.modal.step3."); // Somewhere near the top of the 3rd step component code

// This is what you should see in most places
const buttonCopy = t("confirm-btn");              // "подтвердить"
```
**NOTE:** example is in JS but probably need a similar toolset for the backend
