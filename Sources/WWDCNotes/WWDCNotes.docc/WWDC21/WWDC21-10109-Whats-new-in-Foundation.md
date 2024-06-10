# What's new in Foundation

Discover how the latest updates to Foundation can help you improve your app's localization and internationalization support. Find out about the new AttributedString, designed specifically for Swift, and learn how you can use Markdown to apply style to your localized strings. Explore the grammar agreement engine, which automatically fixes up localized strings so they match grammatical gender and pluralization. And we’ll take you through improvements to date and number formatting that simplify complex requirements while also improving performance.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10109", purpose: link, label: "Watch Video (37 min)")

   @Contributors {
      @GitHubUser(zntfdr)
      @GitHubUser(fbernutz)
   }
}



![Sketchnote about what’s new in Foundation at WWDC 2021. It shows news about internationalization and localization improvements, in detail Attributed String, Formatters and Automatic Grammar Agreement.][sketchnote]

## AttributedString

- An attributed string is a combination of:
  - characters
  - a set of ranges
  - a dictionary

- Attributed strings allow you to associate attributes, which are key-value pairs, to a specific range of a string

- New [`AttributedString`][AttributedString] type
  - value type, built from the ground up for Swift
  - same character-counting behavior as Swift `String`
  - localizable
  - built with safety and security in mind

Example:

```swift
var thanks = AttributedString("Thank you!")
thanks.font = .body.bold()

var website = AttributedString("Please visit our website.")
website.font = .body.italic()
website.link = URL(string: "http://www.example.com")

// AttributeContainer - a place you can hold attributes and values on their own without the string
var container = AttributeContainer()
container.foregroundColor = .red
container.underlineColor = .primary

thanks.mergeAttributes(container)
website.mergeAttributes(container)
```

- `AttributedString` views: give us insight of the attribute string, they're Swift collections
  - `characters` - which provides access to the `String` instance
  - `runs` - which provides access to the attributes
    - A run is the starting location, length, and value of a particular attribute
    - we can also filter runs per attribute, e.g. `attributedString.runs[\.link]`, this will separate the attribute string considering solely that specific attribute, and returning `nil` when that attribute is not applied on a run

For example, here's how to highlight all punctuation of a string:

```swift
var message = AttributedString(localized: "...") 
let characterView = message.characters 
for i in characterView.indices where characterView[i].isPunctuation { 
  message[i..<characterView.index(after: i)].foregroundColor = .orange
} 
```

### Localization

- both `AttributedString` and `NSAttributedString` are fully localizable
- in Swift, we can now use string interpolation, like in SwiftUI

```swift
String(localized: "Would you like to save the document “\(document)”?")
AttributedString(localized: "Would you like to save the document “\(document)”?")
```

- Xcode can generate your strings files from these new initializers using the compiler: Turn this on by going to your build settings, look for the localization settings, and turn on `Use Compiler to Extract Swift Strings`

### Markdown

`AttributedString` comes with markdown support, including links and custom attributes:

```swift
This text contains [a link](http://www.example.com).

This text contains ![an image](http://www.example.com/my_image.gif).

This text contains ^[a custom attribute](rainbow: 'extreme').
```

Here's an example of custom attribute and how to parse it:

```swift
// Attribute scopes 
extension AttributeScopes 
  struct CaffeAppAttributes: AttributeScope {
    let rainbow: RainbowAttribute 
    let swiftUI: SwiftUIAttributes 
  }
  var caffeApp: CaffeAppAttributes.Type { CaffeAppAttributes.self } 
}

let header = AttributedString(
  localized: "^[Fast & Delicious](rainbow: 'extreme') Food",
  including: \.caffeApp
)
```

## Formatters

- brand new API
- no need to cache formatter anymore

Date formatter example:

```swift
func formattingDates() {
  // Note: This will use your current date & time plus current locale. 
  // Example output is for en_US locale.
  let date = Date.now

  let formatted = date.formatted() // equivalent to date.formatted(.dateTime) 
  // example: "6/7/2021, 9:42 AM"
  print(formatted)

  let onlyYearDayMonth = date.formatted(.dateTime.year().day().month())
  // example: "Jun 7, 2021"
  print(onlyYearDayMonth)

  let onlyYearDayMonthWide = date.formatted(.dateTime.year().day().month(.wide))
  // example: "June 7, 2021"
  print(onlyYearDayMonthWide)

  let onlyDate = date.formatted(date: .numeric, time: .omitted)
  // example: "6/7/2021"
  print(onlyDate)

  let onlyTime = date.formatted(date: .omitted, time: .shortened)
  // example: "9:42 AM"
  print(onlyTime)
}
```

- all these `formatted(date:)` parameters are called fields
- fields order do not matter
- all fields have a default for the shortest versions of the API

Formatters work great with `AttributedString`, here's how to format a date and color just the week day of the string:

```swift
import SwiftUI

struct ContentView: View {
  @State var date = Date.now
  @Environment(\.locale) var locale

  var dateString : AttributedString {
    var str = date.formatted(
      .dateTime.minute().hour().weekday()
      .locale(locale)
      .attributed
    )

    let weekday = AttributeContainer
      .dateField(.weekday)

    let color = AttributeContainer
      .foregroundColor(.orange)

    str.replaceAttributes(weekday, with: color)

    return str
  }

  var body: some View {
    VStack {
      Text("Next free coffee")
      Text(dateString).font(.title2)
    }
    .multilineTextAlignment(.center)
  }
}
```

New way to convert strings to values via strategies:

```swift
func parsingDates() {
  let date = Date.now

  let format = Date.FormatStyle().year().day().month()
  let formatted = date.formatted(format)
  // example: "Jun 7, 2021"
  print(formatted)

  if let date = try? Date(formatted, strategy: format) {
    // example: 2021-06-07 07:00:00 +0000
    print(date)
  }
}

// Work with String interpolation:
func parsingDatesStrategies() {
  let strategy = Date.ParseStrategy(
    format: "\(year: .defaultDigits)-\(month: .twoDigits)-\(day: .twoDigits)",
    timeZone: TimeZone.current)

  if let date = try? Date("2021-06-07", strategy: strategy) {
    // date is 2021-06-07 07:00:00 +0000
    print(date)
  }
}
```

## Automatic Grammar Agreement

Makes localization easy when a certain sentence need to change based on the quantity and gender of the objects, we only need to provide the singular word, Foundation will take care of plurals and more.

Example:

```swift
func addToOrderEnglish() {
  // Note: This will use your current locale. Example output is for en_US locale.
  let quantity = 2
  let size = "large"
  let food = "salad"

  let message = AttributedString(localized: "Add ^[\(quantity) \(size) \(food)](inflect: true) to your order")
  print(message)
}
```

These are all the localization strings needed:

```xml
// MARK: en.strings

"Add ^[%lld %@ %@](inflect: true) to your order" = "Add ^[%lld %@ %@](inflect: true) to your order"; 

"Pizza" = "Pizza";
"Juice" = "Juice";
"Salad" = "Salad";

"Small" = "Small";
"Large" = "Large";

// MARK: es.strings

"Add ^[%lld %@ %@](inflect: true) to your order" = "Añadir [%1lld %3$@ %2$@](inflect: true) a tu pedido"; 

"Pizza" = "Pizza";
"Juice" = "Jugo";
"Salad" = "Ensalada";

"Small" = "Pequeño";
"Large" = "Grande";
```

[AttributedString]: https://developer.apple.com/documentation/foundation/attributedstring
[sketchnote]: https://fbernutz.github.io/images/sketchnotes/wwdc21-whats-new-foundation.jpg
