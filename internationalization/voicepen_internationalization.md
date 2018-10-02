# Voicepen

You can use our internal voicepen command-line tool to rapidly speed up your internationalization. Here's how.

## Prerequisites

Before using voicepen, we recommend internationalizing one string by hand
so that you're familiar with a bit of what's going on behind the hood. Check
out the [internationalization readme](/asana2/docs/luna2/web_app/luna-ui/internationalization.md).

## Quick Start

Voicepen will run against your diff of changed files by default.

If you have this:
```typescript
// myfile.ts
export interface Services {
}
export interface Props {
  services: Services;
}
class Component extends PureComponent<Props, State> {
  render() {
    return "Task"
  }
}
```

First, add your `tx` invocations by hand, like so:
```typescript
//myfile.ts
export interface Services {
}
export interface Props {
  services: Services;
}
class Component extends PureComponent<Props, State> {
  const _tx = this.props.services.localization.tx;
  render() {
    return this._tx("Task");
  }
}
```

* Note we only added the shorthand variable declaration and the tx call itself. 
* Please **don't rename tx** to anything else. Voicepen looks for calls to a function called `tx`/`_tx` when doing it's magic.

Next, go to your command line and just type `voicepen` to get started:
```
$ voicepen
Running voicepen on path: myfile.ts
Message:  Task
	This message has only one word.
	It would be good to add some clarity to help our translators out.
	What's the part of speech?
	Please enter some word(s) to clarify (noun, verb, adj): adj
	OK. Using the key:  Task [adj]
Your key is: Task [adj]
Adding key to en.yaml...Done.
Opening your en.yaml file...
Updated the entries in the en.yaml file.
All done!
```

Boom! It's like magic. You don't even have to update your source code - just sit back and wait for sand to build.

## What Happened Here?

Voicepen ran against all files that have been changed relative to your personal-master branch. Then it added imports and keys to en.yaml when necessary. 
Along the way, you will receive prompts when we think we need more information or clarification from you. It should hopefully be self explanatory, but if it isn't, ask in the `#internationalization` channel. 

## Other usage modes

If you want to run voicepen only against one path then do this:
```
$ voicepen --path path/to/file
```

And if you only want to add one key at a time, do this
```
$ voicepen --add
Enter your message below
> Your message here
Your key is: Your message here
Adding key to en.yaml...Done.
Opening your en.yaml file...
Updated the entries in the en.yaml file.
All done!
```

Note that in `--add` mode, you have to manually change your source code if the final key differs from the initial message. If you're going to do this (but why would you), make sure to do a find/replace in your code for the final key. 
HIGHLY recommend using `--path` mode instead, as it also gives location and doesn't require the extra hassle.

# Features

## Magically import LocalizationService in Luna2

No need to look up the import statement for localization service - voicepen will put it in your file for you.

* Make a change to the file you're internationalizing (add some tx invocations)
* Run voicepen
* Voicepen will detect changes to your file and add the import statement for you

## Automatically add localization to your Services type

Voicepen will see Service declarations and attempt to add localization to them if there isn't on already:

Before:
```typescript
export interface Services {
    logging: LoggingService;
}
```

After:
```typescript
export interface Services {
    logging: LoggingService;
    localization: LocalizationService;
}
```

## Automatically import the `tx` function in Luna1

In Luna1, you import a `tx` function directly instead of using LocalizationService. Voicepen will
import the tx function for you.

* Make a change to the file you're internationalizing (add some tx invocations)
* Run voicepen
* Voicepen will detect changes to your file and add the import statement for you

## Let voicepen remind you when to disambiguate

English has a funny property in that most words can be either a noun or verb without changing their spelling. For example, "Foot" could mean a noun, as in the body part, or a verb, as in "foot the bill."

This becomes a problem in translation, because in most other languages the verb- and noun- forms of words are usually spelled differently. We need to tell our translators which is which.

That's why voicepen will automatically prompt you for disambiguation when you internationalize strings with a single word.

```
$ voicepen
Running voicepen on path: myfile.ts
Message:  Project
	This message has only one word.
	It would be good to add some clarity to help our translators out.
	What's the part of speech?
	Please enter some word(s) to clarify (noun, verb, adj): noun
	OK. Using the key:  Project [noun]
Your key is: Project [noun]
Adding key to en.yaml...Done.
Opening your en.yaml file...
Updated the entries in the en.yaml file.
All done!
```

## Voicepen will update your source code after you add disambiguation

Square-bracket disambiguations result in a change to your translation key. You would normally need to go back to your source code to update the key, but when you disambiguate with voicepen, it will do this step for you.

1. Write your key without disambiguation:
```
tx("Fly");
```

2. Run voicepen, disambiguate "Fly" as a verb

3. Check your diff:
```
tx("Fly [verb]");
```

Caveat: `--path` mode or `default` mode only, not in `--add` mode. 

## Voicepen will indicate you as the internationalizer in the dictionary

Sometimes, we'll start the process of translating our text, and translators will have
questions about what a phrase means. That's why voicepen puts down your username
in the dictionary entry each time you use it.

This makes it easy for Asanas (both engineers and non-engineers) to know who to talk
to when they have a question about a string.

## Save the current filepath and line number location of the string with voicepen

This will be visible in en.yaml after you run voicepen.

Note that these file paths are for reference only - they have no impact whatsoever
on how the dictionary entry is looked up at runtime. They are not a source of truth.
Instead, they are meant to be a convenient way for humans to look up where a string
was used at some point in the past.

This is especially useful if there are questions about a string later.

## Give voicepen your clarifying comments

If your string is a complete sentence, voicepen will assume that it's clear enough
on its own. However, shorter strings, especially button text and partial phrases,
can be very vague to translators.

Voicepen will prompt you for a comment if it detects that the string is vague. Your
short comment will be saved in the dictionary as metadata and be made visible to our
translators to help the produce a more accurate translation.

If voicepen doesn't ask you for a comment, you're more than welcome to add one
to the dictionary entry in en.yaml yourself anyway.

## Voicepen will deal with the ellipsis character at the end of strings for you

We love seeing the  ellipsis character (…) throughout the product because it's more typographically correct. 
But we sure hate copying into the codebase each time we want to use it! So, voicepen will automatically
find all sets of three periods (...) and convert them into the ellipsis character for you in the dictionary.

The translation _key_ will always be kept as three periods to keep the source code easy-to-type. But the
translation _value_ will contain the ellipsis character.

Note that voicepen will only pick up on triple-dots at the _end_ of strings, which is the most common case.

## Automatically truncate keys that are long

In some cases you may end up with:
```
tx("some really really long string that is so long it can't even fit on one line and it takes up so much space in your code and it's really really big")
```

Voicepen will truncate long keys to 12 words followed by an `[...]` to indicate there's more to the key. 
It makes your code much easier to read and you don't have to deal with pesky multiline strings and such. 
