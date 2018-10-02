# Internationalization

Internationalization \(or i18n for short\) is the process of making your code translatable into multiple languages.

Related: [voicepen](voicepen_internationalization.md), [i18n in Luna1](internationalization_luna1/), [i18n Best Practices](internationalization_best_practices.md)

Let's get started.

## Luna1

See [internationalization in Luna1](internationalization_luna1/).

## Luna2 Quick Start

In LunaUi, use `LocalizationService` to display localized text:

```typescript
import LocalizationService from "asana/web/services/localization_service";

export interface Props {
    services: {
      localization: LocalizationService;
    }
}

export class Component extends PureComponent<Props, {}> {
    render() {
      return r.div({}, [
        // This will look up the entry for "Welcome to Asana!" in the current
        // language dictionary and return it as a string.
        this.props.services.localization.tx("Welcome to Asana!"),

        // This will look up the entry for "Archive [noun]", which happens to be "Archive".
        // It's best to disambiguate single words by putting its part of speech in square brackets.
        this.props.services.localization.tx("Archive [noun]"),

        // Look up this entry and insert "Acme Corporation" into it where the curly braces are.
        this.props.services.localization.tx("You are a member of {organizationName}.", {
          organizationName: "Acme Corporation"
        }),

        // Look up the correct pluralized key to use when `count` is 5.
        // Use the "{count}" parameter when you want to pluralize.
        this.props.services.localization.tx("You have {count} Tasks remaining.", {
          count: 5
        });

        // Capture text for decoration. You do not need to do this if you are trying to 
        // put a react node into the text - use curly braces for that.
        this.props.services.localization.tx("You have <b>bold text</b>.", {
          b: (s: string) => r.strong({}, [s]) 
        });

        // Look up the correct version of a string to render when `variant` is "task".
        // You might want to do this if you're interpolating a string that isn't user generated.
        // The grammar for this sentence might change based on what "variant" is, so you
        // might not want to use a regular interpolation.
        // If the variant is missing, tx will fall back to normal interpolation.
        this.props.services.localization.tx("You liked this {variant}", {
          variant: "task"
        });
      ]);
    }
}
```

Then, update our global [en.yaml](https://github.com/ccankovjunk/docs-test/tree/abeb76228f71126980072272dd2f38080ca159bf/asana2/asana/internationalization/translations/en.yaml) file with the new text:

```yaml
Welcome to Asana!: Welcome to Asana!
Archive [noun]: Archive
You are a member of {organizationName}.: "You are a member of {organizationName}."
You have {count} Tasks remaining:
  count_one: "You have 1 Task remaining"
  count_other: "You have {count} Tasks remaining"
You have bold text.: You have bold text.
You liked this {variant}:
  variants:
    task:
      You liked this task.
    comment:
      You liked this comment.
    _default:
      You liked this {variant}.
```

Your view will now display the English version. You can now check in and deploy your code - there's no need to wait for translation to be complete before turning on your feature.

### PROTIP: Use [voicepen](voicepen_internationalization.md) to add strings quickly

You're welcome to add strings to `en.yaml` manually. However, we recommend using [the `voicepen` command](voicepen_internationalization.md) to do it for you automatically. It's available from your command line like so:

```text
$ voicepen
Enter your new message below and hit Return when done:
> Hello, World!
Your message is: Hello, World!
Adding your new message...
Opening your en.yaml file...
Sorting the entire en.yaml file...
All done!
```

You can read more in-depth docs in [the voicepen readme](voicepen_internationalization.md).

## Implementation details

### Adding localization\_service as a dependency in your bazel BUILD file

Seeing this compilation error?

```text
Cannot find module 'asana/web/services/localization_service'.
```

Add this dependency to your BUILD file:

```text
"//asana/web/services:localization_services"
```

For example, if your existing web\_library is "my\_cool\_library", then you would add localization\_service to the `deps` list:

```text
web_library(
    name = "my_cool_library",
    srcs = srcs,
    deps = (
        "//asana/web/services:localization_services"
    ),
    exports = exports)
```

### Translated text is available through LocalizationServce.

In case you are wondering whether you need to add a locale or a language, `LocalizationService` always uses the correct locale to display to the current user. You just need to import it like any other service and use its `tx` method to get strings from it.

```text
import { LocalizationService } from "asana/web/services/index";

export type Services = {
    localization: LocalizationService;
};

export interface Props {
    services: Services;
}

export class Component extends PureComponent<Props, {}> {
    render() {
        return r.div({}, [
            this.props.services.localization.tx("Hello, World");
        ];
    }
}
```

### Alias tx to save typing

If you're internationalizing multiple strings in the same function, you can save some typing by aliasing `tx` to a variable at the top of the function:

```text
export class Component extends PureComponent<Props, {}> {
  render() {
    const tx = this.props.services.localization.tx;
    return r.div({}, [
      tx("Foo"),
      tx("Bar"),
      tx("Baz")
    ];
  }
}
```

## Writing good keys for `tx`:

It's important to understand that `TranslationKey`s are separate from their content:

```text
// ⛔️ Avoid actually doing this, it's not readable.
this.props.services.localization.tx("message1");
// returns "Some random message!"
```

However, we strongly recommend making these keys as identical as possible to the content being returned in English:

```text
// ✅ Do it this way, it's readable.
this.props.services.localization.tx("You are really great!");
// returns "You are really great!"
```

PS: we've put some good default behavior into voicepen - it will do proper truncation and disambiguation for you.

### Variables and Interpolation

If you need to interpolate user data or a component into a string, use the curly brace variables:

```text
tx("Hello, {userName}", {userName: user.name()});
```

If `user.name()` is a string, you'll get a string back. If you try to insert a component or a DOM node, though:

```text
tx("Hello, {userName}", {userName: r.a({href: user.url()}, user.name())});
```

Then you will get an array like `["Hello ", r.a({href: user.url()}, user.name())]` back instead.

### Placeholders and Capturing

Some strings require styling to be applied to the string itself. Bold is a common example:

```text
[r.strong({}, "Create tasks"), "in asana by pressing Enter"]
```

You can't i18n these separately, because `create tasks` could appear anywhere in the translated text \(word order is not guaranteed\). Instead you should capture the text you are interested in using angle braces and then use a callback to wrap the translated version:

```text
tx("<b>Create tasks</b> in asana by pressing Enter", {b: (s: string) => r.strong({}, s)})
```

One point of confusion sometimes is when to use this versus a curly brace variable. If you are trying to internationalize something where you are feeding data that doesn't depend on the string text itself \(so any kind of component qualifies\) then use curly braces. If you need to wrap the text of the string itself \(much like the angle bracket tag in the key\) then use placeholders.

Another way to think about it is that `"<tag></tag>" == "{tag}"` \(so capturing nothing is the same as using a curly brace variable\).

Note: do NOT modify the string given to the callback function in ANY WAY. There's legitimately no way to know what's going to be in there, or what's correct for a given language. Even simple things like capitalization do not translate across languages. If you think you need to modify the callback string, talk to the i18n team.

### Use variants when placeholders are translated words

As mentioned before, placeholders can be used to interpolate variables in a string:

```text
// this is OK.
"You renamed the task to {newName}"
```

This is a proper usage of placeholders, because `newName` is User-Generated Content. Here's a less-good idea:

```text
// avoid this!
"You renamed the {thing}."
```

Here, "thing" could be a few things: "Task", "Conversation", "Project", etc. Each of these words would need to be translated before being interpolated into the sentence, which is a bad sign. In the above example, the word "the" needs to be translated differently depending on the value of `{thing}`. This isn't an issue in English because articles are not gendered, but that is not true in other languages.

```text
// this is great!
"You renamed the {variant}"

// en.yaml
You renamed the {variant}:
  variants:
    _default: You renamed the {variant}
    project: You renamed the project
    task: You renamed the task
```

Like `{count}`, the placeholder `{variant}` is a magic word that triggers special logic in our system. A `{variant}` is a placeholder that expects one of a certain set of values. Each possible value becomes an independently-translated key. This gives our translators complete freedom to change the entire sentence depending on the value of `{variant}`.

Variants are designed to expect a hardcoded set of values, but the `_default` value is provided as a fallback for when an unexpected variant value appears \(for example, "conversation" in the dictionary above\). Whenever a variant falls back to the default, a warning is fired, reminding you to update your set of expected variants. Fixing these warnings is important to make sure that your translations make grammatical sense in each language.

### Use square brackets to disambiguate duplicates

When the key is ambiguous, you may want to include extra context at the end of the key inside of square brackets. We have a convention of square brackets representing content in a key that won't be present in the referenced message:

```text
// ✅ Do it this way, it's both readable and easy to translate.
this.props.services.localization.tx("Archive [noun]");
// returns "Archive"
```

We recommend always including this square bracket to clarify the Part of Speech whenever your message is a single word. Why? The word "Archive" in English is the same whether you mean the verb or the noun - but this is rarely the case in other languages. It's best to always be clear about which you mean when you're translating a single word.

### We begin translation once your text hits next-master

Once you check in your newly internationalized code, what happens next?

* Once your code is merged into next-master, we'll notice the new entries in [en.yaml](https://github.com/ccankovjunk/docs-test/tree/abeb76228f71126980072272dd2f38080ca159bf/asana2/asana/internationalization/translations/en.yaml).
* We'll then request your text to be translated into each of our supported languages.
* The new translations will then be added to language-specific files such as [fr.yaml](https://github.com/ccankovjunk/docs-test/tree/abeb76228f71126980072272dd2f38080ca159bf/asana2/asana/internationalization/translations/fr.yaml) within a few days.

### Tests are NOT needed

You generally do **not** need to add tests around internationalization when using the framework. We have unit tests for the localization framework, as well as integration tests that use the framework in a variety of different contexts.

#### Luna2 testing \(if necessary\)

```text
let services: ServicesMock.Services;

test("Uses French localization in some component", () => {
    const localization = LocalizationMock.create("fr");
    services = ServicesMock.create({
        localization: localization
    });
    const component = Enzyme.shallow(ExampleComponent.create({}));
    ...  
});
```

### Luna1 testing \(if necessary\)

```text
suite.test("Uses French localization somewhere", function(step) {
  localization_testing.runWithLocaleInTest(step, "fr", function() {
    step.onServerHandlerThenSync(function(t) {
        ... 
    });
    step.onClient(function(t) {
      assert.eq(
        "Francais",
        UiTesting.elementWithClass("some-class").textContent());
    });
  });
});
```

### How to create new users in a different locale

Note that this approach will work for real languages that we're supporting like French and German, but will _not_ work for pseudolocales such as Accents and UI Expansion.

* Get an email address to test with. Use [Mailinator](http://mailinator.com) if you don't have one.
* Run this curl command \(make sure to put your test email in there\):

```text
curl -X POST https://app.asana.com/-/signup -F 'lang_pref=de' -F 'email=EMAIL_HERE'
```

* You should get an email in German. Click on its link to enter the NUX flow in German and proceed from there.
* Use `lang_pref=fr` for French.

_Using NUX experiments dashboard_

This approach lets you change languages on the first screen of NUX, and also allows you to use psuedolocales like UI Expansion.

* Go to [NUX experiments dashboard](https://app.asana.com/app/nux_experiments_dashboard)
* Turn on `i18n_dogfooding` and `i18n` flags.
* Use either the direct login link or the email link to proceed to NUX.
* The locale picker will display at the bottom of the NUX screen.

### How to modify text that's already been translated

Let's say you want to modify some text that's already translated:

```text
// in your en.yaml
Sign up [verb]:
  value: Sign up

// in your code
tx("Sign up [verb]")
```

If you simply create a new key, there will be missing translations:

```text
// in your en.yaml
Sign up [verb]:
  value: Sign up
Sign up now [verb]:
  value: Sign up now

// in your code
tx("Sign up now [verb]")
// French/German users will see this in English until it's translated!
```

But you can define a fallback key to avoid this:

```text
// in your en.yaml
Sign up [verb]:
  value: Sign up
Sign up now [verb]:
  value: Sign up now
  fallback_key: Sign up [verb]

// in your code
tx("Sign up now [verb]")
// French/German users will see the translation of the old key until the new one is translated!
```

Notice that you've created a _second entry_ in en.yaml which refers to the old entry via `fallback_key`.

So, step-by-step:

* Duplicate the entry you want to modify in en.yaml
* Modify the key and value of the new entry however you want it to look
* Add a `fallback_key` to the new entry pointing to the old one
* Use the new key in the code with confidence that it's already sort of translated!

You should now see that your text still displays in German in sand, even though the English version has been changed. You're done!

### How to ship internationalized code without waiting for translations

We work to have most internationalized text translated within a day or so. What if you want to turn your feature on right now?

1. Internationalize your text normally
2. Restrict your feature to English users only, like so

   ```text
   return this.props.services.localization.config.hasInstantTranslations &&
   this.props.services.experiments.enabled("my_flag_name", true);
   ```

3. Note that this will hide your feature from any users with non-english locales.
4. Merge immediately, deploy, and turn your feature on.
5. Later, you can remove the `hasInstantTranslations` check if you want to launch globally.

### How to internationalize images

Overally, we _highly_ recommend that you avoid this problem altogether by never using images containing text. However, this is sometimes unavoidable. In those cases, we recommend putting the static asset URL into the dictionary like so:

```text
// in code
IMG({href: tx("[animated gif of multi select being used]")});

// in en.yaml
"[animated gif of multi select being used]":
    value: https://url.to/image.jpeg
    comment: Upload a new image for this value.
```

This allows non-engineers to upload new images for each new language without further intervention on your part.

## I need to translate text on a soy template!

[There's a separate doc with instructions for that](https://github.com/ccankovjunk/docs-test/tree/abeb76228f71126980072272dd2f38080ca159bf/asana2/docs/luna2/web_app/luna-ui/internationalization_soy_templates.md).

## I have questions!

We have answers! Please reach out the \#internationalization team on Slack if you need anything at all.

