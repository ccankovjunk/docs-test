# Internationalization Best Practices

**Too much stuff to read?** Try our [interactive Best Practices training](https://goo.gl/forms/p9E64GtpQgpv5uiS2) instead!

![Earth](https://media.giphy.com/media/l3V0megwbBeETMgZa/giphy.gif)

Write your code for clear, well-written translations.

Once you know the basics of [how to use our i18n framework](https://github.com/ccankovjunk/docs-test/tree/ec64d54dbe01f87d81c1fb709ba61401e6e9fe8b/lunaui/internationalization/README.md), it's important that you use it in a way that makes it possible to write great translations.

That said, it's not always possible to follow all of the best practices, so use your best judgment when following these.

## 1 - Don't split up sentences.

Avoid splitting up sentences amongst several dictionary entries. Instead, strive to put the complete sentence \(or sentences\) as a single key.

\(This doesn't apply to things such as buttons which are intended to be displayed as single words or phrases.\)

For example, don't do this::

```text
tx("If you leave ") + projectName + tx(", then you won't be able to rejoin.");
```

This results in two separate keys being placed in the dictionary. It's then very difficult for the translator to, say, move the word "rejoin" to the front of this sentence - or even know what the full sentence is in the first place.

Do this instead:

```text
tx("If you leave {projectName}, then you won't be able to rejoin.", {projectName: projectName});
```

## 2 - Put the part of speech in the key when internationalizing single words.

English is weird. Compare the word "comment" across these sentences:

```text
English: You left a comment, so I'll comment, too.
German: Du hast einen Kommentar hinterlassen, also werde ich auch kommentieren.
French: Vous avez laissé un commentaire, alors je vais commenter aussi.
Spanish: Usted dejó un comentario, así que voy a comentar, también.
```

In English, the word "comment" is the same whether it's used as a noun or a verb. In most other languages, there is a slightly different verb- and noun- form.

That's why doing this is a mistake:

```text
function getButtonText() {
  return tx("Comment");
}
```

In this case, the translator can't tell if you mean "Comment, the noun" or "Comment, the verb." If they choose the noun "Kommentar" in German, then the button will mean "display comment", rather than "leave a comment".

The correct way to internationalize this function is like so:

```text
function getButtonText() {
  return tx("Comment [verb]");
}
```

If you use [voicepen](voicepen_internationalization.md), it will automatically ask you for a Part of Speech when you try to add a single word to the dictionary. Use this feature!

**ProTip** - Can't decide whether a word is being used as a verb or a noun? Use this trick: If the text means the same thing with the word "now" added to the end, then it's a verb. If it means the same thing with "Go to" or "This is a" at the beginning, it's a noun. You can also mark things as other parts of speech, or make up your own disambiguation word for clarity.

## 3 - Don't use your own pluralization logic.

For example, this is a bug:

```text
numTasks === 1 ? tx("Task") : tx("Tasks")
```

This is also a bug:

```text
pluralizeWordOnly("Task", numTasks) // not an internationalized plural function!
```

These won't work because some languages \(such as French, Spanish, and Russian\) have **different pluralization rules**. In French, you use the singular form for both zero and one. Russian has three plural forms, and Arabic has six.

So save yourself the trouble and let `tx` pluralize for you:

```text
tx("{count} Tasks", {count: numTasks});
```

Don't forget that `count` is the magic word for pluralization in `tx`!

## 4 - Add a clarifying comment if you must add a sentence fragment.

As we said in point 1, don't split up sentences. But sometimes there's just no way to avoid it. When that's the case, add a **comment** to the dictionary entry to help your translators out.

```text
tx("You have joined ") + teamName + tx(" as a full member!")
```

And add comments to the dictionary entries like so:

```text
"You have joined ":
  value: "You have joined "
  comment: "You have joined (TEAM_NAME as a full member!)"
" as a full member":
  value " as a full member"
  comment: "(You have joined TEAM_NAME) as a full member!"
```

These comments would be enough to communicate the intention of the sentence fragment, although the translators would still not be able to move the words around in the sentence if they choose.

## 5 - Always consider adding a brief comment entry in the dictionary.

Even if your entry is a complete sentence, you're always welcome to add a comment to the entry in the en.yaml file with your string. Take a look at the sentence and consider whether its meaning is clear in isolation.

No need to go overboard, though - one or two sentences is almost always enough.

## 6 - Only interpolate variables that are truly dynamic.

For example, **don't** do this:

```text
tx("You have joined the {thing}", {thing: Vocab.Organization});
```

Since the word "Organization" is basically hardcoded here, it's better to put it in the string:

**do this** instead:

```text
tx("You have joined the {variant}")
```

Using the special placeholder variable `{variant}` triggers the variant system in our framework. This makes it possible for our translators to create article-noun agreement \(like choosing `die`, `das`, or `der` in German\).

Check out more details on how to use variants [here](https://github.com/ccankovjunk/docs-test/tree/ec64d54dbe01f87d81c1fb709ba61401e6e9fe8b/lunaui/internationalization/README.md#use-variants-when-placeholders-are-translated-words).

**this is OK:**

```text
tx("Welcome, {name}!", {name: user.name()});
```

Since the user's name is User-Generated Content, you have no choice but to interpolate it in like this. And we encourage you to do so.

## 7 - Don't start or end entries with whitespace.

For example, don't do this:

```text
tx("Welcome to Asana! ") + tx("We're so glad you're here."); // Avoid this!
```

It's really hard for translators to see this trailing whitespace, and they'll often forget to add it, resulting in these two sentences getting smushed together.

Better:

```text
tx("Welcome to Asana!") + " " + tx("We're so glad you're here.");
```

Also good:

```text
tx("Welcome to Asana! We're so glad you're here.");
```

## 8 - Don't wait to internationalize.

When writing new code, it's best to immediately wrap your user-facing strings in `tx` as soon as you write them.

Wrapping ASAP makes it a lot easier to implement these best practices from the get-go, and to get early feedback from pseudolocales and translators. Don't worry about requesting translations too early - for product text, it's relatively cheap!

