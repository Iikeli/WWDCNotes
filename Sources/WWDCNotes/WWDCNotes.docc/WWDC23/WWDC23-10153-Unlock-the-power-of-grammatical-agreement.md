# Unlock the power of grammatical agreement

Discover how you can use automatic grammatical agreement in your apps and games to create inclusive and more natural-sounding expressions. We’ll share best practices for working with Foundation, showcase examples in multiple languages, and demonstrate how to use these APIs to enhance the user experience for your apps.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10153", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## Grammatical agreement

- Use `^[Bienvenido](inflect: true) a tu iPhone` to provide automatic inflection (in supported languages like Spanish, French, or Italian)
- New locales added: European Portuguese and German

## Dependency agreement

- New API on localization options named `concepts` to pass a distinct string to another string which are related (e.g. ingredients of a food)

```Swift
var options = AttributedString.LocalizationOptions()
options.concepts = [.localizedPhrase(food.localizedName)]

let size = AttributedString(localized: "small", options: options)
```

- To choose the concept, provide a 1-indexed value like in `^pequeno](agreeWithConcept: 1)`
- The new `agreeWithConcept` option can be used even in older OS versions – it'll be ignored in them

![][agreeWithConcept]

[agreeWithConcept]: WWDC23-10153-agreeWithConcept

- When multiple inflections in one text needed but only one gets `%@` argument passed, use new `agreeWithArgument: 1` instead on other

![][agreeWithArgument]

[agreeWithArgument]: WWDC23-10153-agreeWithArgument

![][agreeWithComparison]

[agreeWithComparison]: WWDC23-10153-agreeWithComparison

- To make demonstrative adjectives grammatically agree in French, use `^[Ce %@](inflect: true) conteient : %@.`

## Inclusive language

- New type `TermOfAddress` for `.masculine`, `.feminine`, or `.neutral` – can be passed as localization option in `.concepts` array

![][neutral]

[neutral]: WWDC23-10153-TermsOfAddress.neutral

- New `referenceConcept` attribute in agreement, like `"\(person.name) is on ^[their](referentConcept: 1) way."`
- You can provide custom `TermOfAddress` using `TermOfAddress.localized(language:pronouns:)` and pass array of `Mophology.Pronoun`

![][LocalizationOptions]

[LocalizationOptions]: WWDC23-10153-LocalizationOptions
