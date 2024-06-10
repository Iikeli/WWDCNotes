# What's new in SwiftUI

There’s never been a better time to develop your apps with SwiftUI. Discover the latest updates to the UI framework — including lists, buttons, and text fields — and learn how these features can help you more fully adopt SwiftUI in your app. Find out how to create beautiful, visually-rich graphics using the canvas view, materials, and enhancements to symbols. Explore multi-column tables on macOS, refinements to focus and keyboard interaction, and the multi-platform search API. And we’ll show you how to take advantage of features like Swift concurrency, a brand new AttributedString, format styles, localization, and so much more.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10018", purpose: link, label: "Watch Video (40 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## AsyncImage

New view that automatically downloads and displays images, also has placeholder, images can be customized via modifiers as usual, can have custom behaviour for error handling.

```swift
AsyncImage(url: ...) { image in
  image
    .resizable()
    .aspectRation(contentMode: .fill)
}
```

## task(_:) concurrency view modifier

`task(_:)` lets you attach an `async` task to the lifetime of your view: it will be triggered when its view appears, and will be cancelled when this view disappears.

```swift
Text(displayValue)
  .task {
    var results = TextProcessResults()
    for try await line in textURL.lines() {
      results.accumulateResults(line: line)
    }
    displayValue = results.textSummary()
  }
```

## Lists & Grids

### Pull to refresh 

Pull to refresh via [`refreshable(action:)`][refreshable(action:)] concurrency view modifier, this modifier configures a refresh action ([RefreshAction][RefreshAction]) and passes down through the environment.

Use an `await` expression inside the action. SwiftUI shows a refresh indicator, which stays visible for the duration of the awaited operation.

```swift
List(mailbox.conversations) {
  ConversationCell($0)
}
.refreshable {
  await mailbox.fetch()
}
```

### Binding

New `List` and `ForEach` initializers allowing us to get a binding per each element:

```swift
struct DirectionsList: View {
  @Binding var directions: [Direction] 

  var body: some View { 
    List($directions) { $direction in 
      Label { 
    TextField("Instructions", text: $direction.text)
    } icon: { 
     DirectionsIcon(direction) 
    }
  }
  }
}
```

This is back-ported all the way to iOS 13.

### Separator Customization

- [`listRowSeparatorTint(_:edges:)`][listRowSeparatorTint(_:edges:)] - custom row separator colors
- [`listSectionSeparatorTint(_:edges:)`][listsectionseparatortint(_:edges:)] - custom section separator colors
- [`listRowSeparator(_:edges:)`][listRowSeparator(_:edges:)] - can be used to hide the separators altogether

### Swipe Actions

New [`swipeActions(edge:allowsFullSwipe:content:)`][swipeActions(edge:allowsFullSwipe:content:)] view modifier to add swipe actions.

Define each action with `Button`s, use the `tint(_:)` view modifier to customize the background color (or use the button's role).

```swift
List(store.messages) { message in
  MessageCell(message: message)
  .swipeActions(edge: .leading) {
    Button { store.toggleUnread(message) } label: {
      if message.isUnread {
        Label("Read", systemImage: "envelope.open")
      } else {
        Label("Unread", systemImage: "envelope.badge")
      }
    }
    .tint(.yellow)
  }
  .swipeActions(edge: .trailing) {
    Button(role: .destructive) {
      store.delete(message)
    } label: {
      Label("Delete", systemImage: "trash")
    }
    .tint(.blue)
    Button { store.flag(message) } label: {
      Label("Flag", systemImage: "flag")
    }
    .tint(.green)
  }
  }
}
```

### Style Updates

All styles now come with a new enum-like syntax:

```swift
List {
  ...
}
.listStyle(.grouped)
```

instead of:

```swift
List {
  ...
}
.listStyle(GroupedListStyle())
```

New (macOS-only) style, which alternates the colors of the rows:

```swift
List {
  ...
}
.listStyle(bordered(alternatesRowBackgrounds: true))
```

## Table (macOS-only)

New `Table` view, supports selection, sorting, and more:

```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData

  var body: some View {
  Table(characters) {
    TableColumn("􀟈") { CharacterIcon($0) }
    .width(20)
    TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
    .width(40)
    TableColumn("Name", value: \.name)
    TableColumn("Powers", value: \.powers)
  }
  }
}
```

## Search

New [`searchable(_:text:placement:)`][searchable(_:text:placement:)] view modifiers, it adds a search field where more appropriate based on the context:

```swift
NavigationView {
  List {
    ...
  }
  .searchable(...)
}
```

## Sharing data

- `onDrag` now comes with a `preview` `View` parameter, letting us customize what view to show when dragging.
- new `importsItemProviders` view modifier makes a view a drop target that accepts item providers
- new `exportsItemProviders` view modifier exposes our app data to external system services

## SF Symbols

- Two new rendering modes:
  - Hierarchical - like monochrome, but automatically adds multiple levels of opacity to really emphasize the key elements of the symbol
  - Palette - gives more fine-grained control over individual layers color with custom fills

- SwiftUI automatically chooses the correct symbol variant to use based on the context, for example a symbol used in the tabbar will use the `.fill` variant.

## Canvas

New view allowing immediate-mode drawing similar to `drawRect` from UIKit or AppKit:

```swift
Canvas { context, size in
  let metrics = gridMetrics(in: size)
  for (index, symbol) in symbols.enumerated() {
  let rect = metrics[index]
  let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint)

  context.opacity = opacity
  let image = context.resolve(symbol.image)
  context.draw(image, in: sRect.fit(image.size))
  }
}
```

We can use `TimelineView` to make our canvas update over time.

## Displaying sensitive data

New modifiers that automatically redact sensitive data when the user is no longer authenticated (for when the phone is locked or similar)

```swift
Image(systemName: favoriteSymbol)
  .font(.title2)
  .privacySensitive(true)
```

## Material (blur)

New blur/vibrancy effects:

```swift
struct ColorList: View {
  var body: some View {
    ZStack {
      ...
    materialOverlay
    }
  }
  
  var materialOverlay: some View {
  VStack {
   Text("Symbol Browser")
    .font(.largeTitle.bold())
   Text("\(symbols.count) symbols 🎉")
    .foregroundStyle(.secondary)
    .font(.title2.bold())
  }
  .padding()
  .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16.0))
  }
}
```

## Preview

We can now preview screens in different orientations:

```swift
struct ColorList_Previews: PreviewProvider {
  static var previews: some View {
  ColorList()
    .previewInterfaceOrientation(.portrait)

  ColorList()
    .previewInterfaceOrientation(.landscapeLeft)
  }
}
```

## Text

- markdown support
- [AttributedString][AttributedString]
- restrict dynamic type size of a text/view via new [dynamicTypeSize(_:)][dynamicTypeSize(_:)] view modifier
- make text selectable or not via [textSelection(_:)][textSelection(_:)] view modifier (macOS only)
- new powerful formatters

## TextFields

- support for prompts, separate from its label, to let users know what kind of content a field is expecting. In macOS, the prompt will be used as the placeholder text.
- `onSubmit(_:)` view modifier to detect when the user submits the text (this replaces the previous `TextField`'s `onCommit` parameter
- `submitLabel(_:)` view modifier to customize the return key action, and to help give users a hint of what kind of action will occur when submitting a field

```swift
struct ContentView: View {
  @State private var activity: Activity = .sample
  @State private var newAttendee = PersonNameComponents()

  var body: some View {
    TextField("New Person", value: $newAttendee,
      format: .name(style: .medium)
    )
    .onSubmit {
      activity.append(Person(newAttendee))
      newAttendee = PersonNameComponents()
    }
    .submitLabel(.done)
  }
}
```

- keyboard toolbar support via the usual `toolbar(_:)` view modifier with new `.keyboard` placement 

```swift
struct ContentView: View {
  @State private var activity: Activity = .sample
  @FocusState private var focusedField: Field?

  var body: some View {
    Form {
      TextField("Name", text: $activity.name, prompt: Text("New Activity"))
      TextField("Location", text: $activity.location)
      DatePicker("Date", selection: $activity.date)
    }
    .toolbar {
      ToolbarItemGroup(placement: .keyboard) {
        Button(action: selectPreviousField) {
          Label("Previous", systemImage: "chevron.up")
        }
        .disabled(!hasPreviousField)

        Button(action: selectNextField) {
          Label("Next", systemImage: "chevron.down")
        }
        .disabled(!hasNextField)
      }
    }
  }

  private func selectPreviousField() {
     focusedField = focusedField.map {
      Field(rawValue: $0.rawValue - 1)!
     }
  }

  private var hasPreviousField: Bool {
    if let currentFocusedField = focusedField {
      return currentFocusedField.rawValue > 0
    } else {
      return false
    }
  }

  private func selectNextField() {
     focusedField = focusedField.map {
      Field(rawValue: $0.rawValue + 1)!
     }
  }

  private var hasNextField: Bool {
    if let currentFocusedField = focusedField {
      return currentFocusedField.rawValue < Field.allCases.count
    } else {
      return false
    }
  }
}
```

- textfield focus control via `@FocusState` property wrapper:

```swift
struct ContentView: View {
  @State private var activity: Activity = .sample
  @State private var newAttendee = PersonNameComponents()
  @FocusState private var addAttendeeIsFocused: Bool = false

  var body: some View {
  VStack(alignment: .leading) {
    TextField("New Person", value: $newAttendee, format: .name(style: .medium))
    .focused($addAttendeeIsFocused)

    ControlGroup {
    Button {
      addAttendeeIsFocused = true
    } label: {
       Label("Add Attendee", systemImage: "plus")
    }
    }
  }
  }
}
```

## Buttons

- New bordered style (`Button("Add") {}.buttonStyle(.bordered)`), which supports tinting via the `.tint` view modifier
- new [`controlSize(_:)`][controlSize(_:)] view modifier for different buttons appearances
- new [`controlProminence(_:)`][controlProminence(_:)] to highlight importance of each button

```swift
struct ContentView: View {
  var body: some View {
    VStack {
      Button(action: addToJar) {
        Text("Add to Jar").frame(maxWidth: 300)
      }
      .controlProminence(.increased)
      .keyboardShortcut(.defaultAction)

      Button(action: addToWatchlist) {
        Text("Add to Watchlist").frame(maxWidth: 300)
      }
      .tint(.accentColor)
    }
    .buttonStyle(.bordered)
    .controlSize(.large)
  }

  private func addToJar() {}
  private func addToWatchlist() {}
}
```

- New Button roles to give each button additional semantics, which SwiftUI uses to display the button accordingly:

```swift
struct ContentView: View {
  var entry: ButtonEntry = .sample

  var body: some View {
  ButtonEntryCell(entry)
    .contextMenu {
    Section {
      Button("Open") {
        // ...
      }
      // This button will have red tint as it's destructive
      Button("Delete...", role: .destructive) {
        // ...
      }
    }
  }
}
```

- Buttons confirmation dialogs via [`confirmationDialog`][cd] view modifier:

```swift
struct ContentView: View {
  var entry: ButtonEntry = .sample
  @State private var showConfirmation: Bool = false

  var body: some View {
    ButtonEntryCell(entry)
      .contextMenu {
        Section {
          Button("Open") {
            // ...
          }
          Button("Delete...", role: .destructive) {
            showConfirmation = true
            // ...
          }
        }
      }
      .confirmationDialog(
        "Are you sure you want to delete \(entry.name)?",
        isPresented: $showConfirmation
      ) {
        Button("Delete", role: .destructive) {
          // delete the entry
        }
      } message: {
        Text("Deleting \(entry.name) will remove it from all of your jars.")
      }
  }
}
```

## Menus

More flexibility and new modifiers to control primary and secondary actions:

```swift
struct ContentView: View {
  var buttonEntry: ButtonEntry = .sample
  @StateObject private var jarStore = JarStore()

  var body: some View {
    Menu("Add") {
      ForEach(jarStore.allJars) { jar in
        Button("Add to \(jar.name)") {
         jarStore.add(buttonEntry, to: jar)
        }
      }
    } primaryAction: {
      jarStore.addToDefaultJar(buttonEntry)
    }
    .menuStyle(BorderedButtonMenuStyle())
    .
  }
}
```

## ControlGroup

New view used to gather controls together (the system will display the controls at the right place with correct spacing etc):

```swift
ControlGroup {
  Button(action: archive) {
    Label("Archive", systemImage: "archiveBox")
  }
  Button(action: delete) {
    Label("Delete", systemName: "trash")
  }
}
``` 

[cd]: https://developer.apple.com/documentation/swiftui/texteditor/confirmationDialog(_:ispresented:titlevisibility:actions:)-90mxz?changes=latest_minor
[controlSize(_:)]: https://developer.apple.com/documentation/swiftui/view/controlsize(_:)?changes=l_5
[controlProminence(_:)]: https://developer.apple.com/documentation/swiftui/view/controlprominence(_:)?changes=l_5
[textSelection(_:)]: https://developer.apple.com/documentation/swiftui/view/textselection(_:)
[dynamicTypeSize(_:)]: https://developer.apple.com/documentation/swiftui/view/dynamictypesize(_:)-1m2tf
[AttributedString]: https://developer.apple.com/documentation/foundation/attributedstring
[searchable(_:text:placement:)]: https://developer.apple.com/documentation/SwiftUI/View/searchable(_:text:placement:)-1r1py
[swipeActions(edge:allowsFullSwipe:content:)]: https://developer.apple.com/documentation/swiftui/section/swipeactions(edge:allowsfullswipe:content:)
[listRowSeparator(_:edges:)]: https://developer.apple.com/documentation/swiftui/section/listrowseparator(_:edges:)
[listsectionseparatortint(_:edges:)]: https://developer.apple.com/documentation/swiftui/section/listsectionseparatortint(_:edges:)
[listRowSeparatorTint(_:edges:)]: https://developer.apple.com/documentation/swiftui/section/listrowseparatortint(_:edges:)
[task(_:)]: https://developer.apple.com/documentation/swiftui/emptyview/task(_:)
[RefreshAction]: https://developer.apple.com/documentation/swiftui/refreshaction
[refreshable(action:)]: https://developer.apple.com/documentation/swiftui/view/refreshable(action:)
