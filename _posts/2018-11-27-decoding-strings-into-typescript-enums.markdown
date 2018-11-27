---
layout: post
title: "Decoding strings into TypeScript enums"
date: 2018-11-27 18:45:04
tags:
  - javascript
  - typescript
  - enum
  - typesafe
  - decode
---

Staying in a relatively safe world of TypeScript code, with strict null checks, is all fine and
dandy. Sometimes though one has to deal with the cruel outside world: JSON from the server or from
the localStorage, user input, route parameters, and so on. Then we have to shoehorn the uncontrolled
chaos from the outside into our nice algebraic type system which strives to making the impossible
states unrepresentable. Here's a simple example: an enumeration of Languages.

{% highlight typescript %}
enum Language {
  DE = "de",
  FR = "fr",
  IT = "it",
  EN = "en"
}
{% endhighlight %}

That's a handy thing to define, because, if you write a function returning some localized text, and
happen to forget about English…

{% highlight typescript %}
function thanks(language: Language): string {
  switch (language) {
    case Language.DE:
      return "Danke";

    case Language.FR:
      return "Merci";

    case Language.IT:
      return "Grazie";

    case Language.EN:
      return "Thanks";
  }
}
{% endhighlight %}

The TypeScript transpiler will complain:

    a.ts:8:40 - error TS2366: Function lacks ending return statement and return type does not include 'undefined'.

    8 function thanks(language: Language): string {
                                           ~~~~~~

That makes sure our pattern matching is exhaustive. Fewer bugs!

We'd certainly want to map stringly-typed `'de'`, `'fr'`, `'it'`, and whatever else (`null`,
`undefined`, `NaN`, `[object Object]`) obtained from the outside, into a member of the `Language`
union, right at our module boundary, before doing anything with the value.

{% highlight typescript %}
function toLanguage(x: any): Language {
  switch (String(x)) {
    case "fr":
      return Language.FR;

    case "it":
      return Language.IT;

    case "en":
      return Language.EN;

    default:
      return Language.DE;
  }
}
{% endhighlight %}

Thus, we fall back to German on any kind of nonsense received.

Quite a bit of repetition here, no? It'll only get worse as the enum grows. We can do better, of
course. Here's a little trick to avoid the repetition, stay type-safe, and never revise the
`toLanguage` function later on, should the enum change.

{% highlight typescript %}
function toLanguage(x: any): Language {
  const key = String(x).toUpperCase();
  const mayBeLanguage: Language | undefined = (Language as any)[key];
  return mayBeLanguage ? mayBeLanguage : Language.DE;
}
{% endhighlight %}

Smart cast to the rescue! Code on.
