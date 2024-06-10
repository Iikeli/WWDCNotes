# Swift Regex: Beyond the basics

Go beyond the basics of string processing with Swift Regex. We'll share an overview of Regex and how it works, explore Foundation’s rich data parsers and discover how to integrate your own, and delve into captures. We’ll also provide best practices for matching strings and wielding Regex-powered algorithms with ease.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110358", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Swift 5.7 String Processing updates

### [`Regex`][regex]

- new type in the Swift Standard Library
- A language built-in Regex literal syntax

### [`RegexBuilder`][regexbuilder]

- `@resultBuilder` API 
- pushes the readability of Regex to a whole new level

### Examples

- Regex from String

```swift
let input = "name:  John Appleseed,  user_id:  100"

/// 👇🏻 Matches `user_id:`, followed by zero or more whitespaces, followed by one or more digits.
let regex = try Regex(#"user_id:\s*(\d+)"#)

if let match = input.firstMatch(of: regex) {
  print("Matched: \(match[0])")  // Matched: user_id:  100 
  print("User ID: \(match[1])")  // User ID: 100
}
```

- Regex from a literal 

If the regex is known at compile time, we can use a shorthand literal:

```swift
let input = "name:  John Appleseed,  user_id:  100"

/// 👇🏻 Same Regex as before, but using a literal.
let regex = /user_id:\s*(\d+)/ 

if let match = input.firstMatch(of: regex) {
  print("Matched: \(match.0)")  // Matched: user_id:  100 
  print("User ID: \(match.1)")  // User ID: 100
}
```

- Regex using [`RegexBuilder`][regexbuilder]

`RegexBuilder` helps a lot with readability

```swift
import RegexBuilder

let input = "name:  John Appleseed,  user_id:  100"

/// 👇🏻 Same Regex as before, but using `RegexBuilder`.
let regex = Regex {
  "user_id:"
  OneOrMore(.whitespace)
  Capture(.localizedInteger)
}

if let match = input.firstMatch(of: regex) {
  print("Matched: \(match.0)")  // Matched: user_id:  100 
  print("User ID: \(match.1)")  // User ID: 100
}
```

## Regex use

Swift `Regex` engine provides multiple algorithms:

- [`firstMatch(of:)`][firstmatch(of:)] - finds the first occurrence of the pattern defined by this Regex
- [`wholeMatch(of:)`][wholematch(of:)] - matches the entire string against a Regex (if the Regex doesn't match the whole string, it will fail)
- [`prefixMatch(of:)`][prefixmatch(of:)] - matches the prefix of a string against a Regex (if the Regex doesn't match the prefix of the string, it will fail)

```swift
let input = "name:  John Appleseed,  user_id:  100"

let regex = /user_id:\s*(\d+)/

input.firstMatch(of: regex)       // Regex.Match<(Substring, Substring)>
input.wholeMatch(of: regex)       // nil
input.prefixMatch(of: regex)      // nil
```

The Swift standard library also adds APIs for Regex-based predication:

- [`starts(with:)`][starts(with:)] - returns true if `prefixMatch(of:)` succeeds
- [`replacing(_:with:)`][replacing(_:with:)] - replace the matched string of the regex with the given value
- [`trimmingPrefix(_:)`][trimmingPrefix(_:)] - removes the `prefixMatch(of:)` match from the string
- `split(separator:)` - splits the string using regex matches as separators

```swift
input.starts(with: regex)       // false
input.replacing(regex, with: "456")   // "name:  John Appleseed,  456"
input.trimmingPrefix(regex)       // "name:  John Appleseed,  user_id:  100"
input.split(separator: /\s*,\s*/)   // ["name:  John Appleseed", "user_id:  100"]
```

Swift `Regex` can also be used in Swift's pattern matching syntax in control flow statements:

```swift
switch "abc" {
case /\w+/:
  print("It's a word!")
}
```

Foundation also has regex support, it can be used for:

- formatters
- parsers

Support for: 

- `Date`
- `Number` (ISO8610, Currency)
- `URL`

Example:

```swift
let statement = """
  DSLIP  04/06/20 Paypal  $3,020.85
  CREDIT   04/03/20 Payroll $69.73
  DEBIT  04/02/20 Rent  ($38.25)
  DEBIT  03/31/20 Grocery ($27.44)
  DEBIT  03/24/20 IRS   ($52,249.98)
  """

let regex = Regex {
  //        👇🏻 Foundation-provided date parser with a custom format
  Capture(.date(format: "\(month: .twoDigits)/\(day: .twoDigits)/\(year: .twoDigits)"))
  OneOrMore(.whitespace)
  OneOrMore(.word)
  OneOrMore(.whitespace)
  //        👇🏻 Foundation-provided currency parser with a domain-specific parse strategy
  Capture(.currency(code: "USD").sign(strategy: .accounting))
}
```

## Regex literal

- A `Regex` literal starts and ends with a slash <kbd>/</kbd>
- Swift infers the correct strong type for it
- strongly typed capturing groups

```swift
/Hello, WWDC Notes!/
// Regex<Substring>

//                    👇🏻 we can give the name year to this capturing group
/Hello, WWDC Notes (?<year>\d{2})!/ // Matches "Hello, WWDC Notes 22!"
// Regex<(Substring, year: Substring)>
```

- support for extended Regex literals `#/ .... /#`
  - allows non-semantic whitespaces
  - can split your patterns into multiple lines

## Transforming capture

- `Capture` with a transform closure
- upon matching, the Regex engine calls the transform closure on the matched substring, which produces a result of the desired type
- the corresponding Regex output type becomes the closure's return type

```swift
Regex {
  Capture {
    OneOrMore(.digit)
  } transform: {
    Int($0)   // Int.init?(_: some StringProtocol)
  }
} 
// Regex<(Substring, Int?)>
```

- `TryCapture` removes optionality from transform
- if the transform returns `nil`, it's considered a non-match (the Regex engine will backtrack and try an alternative path)

```swift
Regex {
  TryCapture {
    OneOrMore(.digit)
  } transform: {
    Int($0)   // Int.init?(_: some StringProtocol)
  }
}
// Regex<(Substring, Int)> 
//                    👆🏻 no longer optional!
```

## Reuse an existing parser

- We can use our own regex matching matching engine with `Regex`, even when it's not Swift
- To do so, we need to create our own parser conforming to [`CustomConsumingRegexComponent`][CustomConsumingRegexComponent]

[CustomConsumingRegexComponent]: https://developer.apple.com/documentation/swift/customconsumingregexcomponent
[regex]: https://developer.apple.com/documentation/swift/regex
[regexbuilder]: https://developer.apple.com/documentation/regexbuilder
[firstmatch(of:)]: https://developer.apple.com/documentation/swift/bidirectionalcollection/firstmatch(of:)
[wholematch(of:)]: https://developer.apple.com/documentation/swift/string/wholematch(of:)-1846c
[prefixmatch(of:)]: https://developer.apple.com/documentation/swift/string/prefixmatch(of:)-39s4z
[starts(with:)]: https://developer.apple.com/documentation/swift/bidirectionalcollection/starts(with:)
[replacing(_:with:)]: https://developer.apple.com/documentation/swift/rangereplaceablecollection/replacing(_:with:maxreplacements:)-7nnjw
[trimmingPrefix(_:)]: https://developer.apple.com/documentation/swift/bidirectionalcollection/trimmingprefix(_:)