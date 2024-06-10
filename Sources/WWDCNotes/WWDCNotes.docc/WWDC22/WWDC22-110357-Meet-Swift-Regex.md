# Meet Swift Regex

Learn how you can process strings more effectively when you take advantage of Swift Regex. Come for concise literals but stay for Regex builders — a new, declarative approach to string processing. We'll also explore the Unicode models in String and share how Swift Regex can make Unicode-correct processing easy.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110357", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



In Swift `Regex` is a struct generic over its `Output`, which is the result of applying the regex itself, including captures.

## Initialization

We can create `Regex` in two ways:

1. literal regex (containing regex syntax in between slash delimiters `/`)
  - syntax is compatible with Perl, Python, Ruby, Java, NSRegularExpression, and many, many others
  - can be created during runtime as well via `try Regex(runTimeString)`
    - will throw if the input contains invalid syntax
    - in this case the type is `Regex<AnyRegexOutput>`, as the types and number of captures won't be known until run-time

```swift
let digits = /\d+/ 
//  👆🏻 type Regex<Substring>
```

2. via regex builders
  - regex literals can be used within builders

```swift
let digits = OneOrMore(.digit)
//  👆🏻 type Regex<Substring>
```

Example:

```swift
// Text to parse:
// CREDIT  03/02/2022  Payroll from employer     $200.23
// CREDIT  03/03/2022  Suspect A           $2,000,000.00
// DEBIT   03/03/2022  Ted's Pet Rock Sanctuary    $2,000,000.00
// DEBIT   03/05/2022  Doug's Dugout Dogs      $33.27

import RegexBuilder
let fieldSeparator = /\s{2,}|\t/
let transactionMatcher = Regex {
  /CREDIT|DEBIT/
  fieldSeparator
  One(.date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt)) // 👈🏻 we define which data locale/timezone we want to use
  fieldSeparator
  OneOrMore {
    NegativeLookahead { fieldSeparator } // 👈🏻 we stop as soon as we see one field separator
    CharacterClass.any
  }
  fieldSeparator
  One(.localizedCurrency(code: "USD").locale(Locale(identifier: "en_US")))
}
```

To capture data from the regex use `Capture`

- we capture strongly-typed values (for `Date`s, numbers)

```swift
let fieldSeparator = /\s{2,}|\t/
let transactionMatcher = Regex {
  Capture { /CREDIT|DEBIT/ } // 👈🏻
  fieldSeparator

  Capture { One(.date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt)) } // 👈🏻
  fieldSeparator

  Capture { // 👈🏻
    OneOrMore {
      NegativeLookahead { fieldSeparator }
      CharacterClass.any
    }
  }
  fieldSeparator
  Capture { One(.localizedCurrency(code: "USD").locale(Locale(identifier: "en_US"))) } // 👈🏻
}
// transactionMatcher: Regex<(Substring, Substring, Date, Substring, Decimal)>
```
