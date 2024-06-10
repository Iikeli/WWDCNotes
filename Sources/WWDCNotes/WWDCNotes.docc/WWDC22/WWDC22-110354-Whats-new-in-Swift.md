# What's new in Swift

Join us for an update on Swift. We'll take you through performance improvements, explore more secure and extensible Swift packages, and share advancements in Swift concurrency. We'll also introduce you to Swift Regex, better generics, and other tools built into the language to help you write more flexible & expressive code.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110354", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(Jeehut)
      @GitHubUser(zntfdr)
      @GitHubUser(fbernutz)
   }
}



![Sketchnote about news in swift from WWDC 22 with community updates, news for swifts packages, performance improvements, concurrency updates and info to expressive swift][sketchnote]

## Community updates

Two new work groups:

- [Swift Website (SWWG)](https://www.swift.org/website-workgroup/)
- [C++ Interoperability](https://forums.swift.org/t/swift-and-c-interoperability-workgroup-announcement/54998)

Year-round mentorships from now on [Swift.org/diversity](http://Swift.org/diversity) has more details.

Cross-platform support:

- support for Linux package formats: new native (experimental) toolchain installers for Amazon Linux 2 and CentOS 7

Swift news:

- dropped dependency on an external Unicode support library, replaced with a faster native implementation
- this has made Swift smaller, useful when Swift is statically linked (e.g., in containerized deployments for the server)
- now used in Apple's Secure Enclave Processor

## Swift Packages Updates

- Trust on first use (TOFU)
  - new security protocol where the fingerprint of a package is now being recorded when the package is first downloaded
  - subsequent downloads will validate this fingerprint and report an error if the fingerprints are different

### Plugins

Command plugins:

- first step in providing more extensible and secure build tools
- can be used for documentation generation, source code reformatting, and more

Example:

```swift
import PackagePlugin

/// You define a plug-in by creating a struct that conforms to the CommandPlugin protocol.
@main struct MyPlugin: CommandPlugin {

  // Add a function that tells your plug-in which tool you'd like to invoke
  func performCommand(context: PluginContext, arguments: [String]) throws {
    let process = try Process.run(doccExec, arguments: doccArgs) // here we call our tool, in this case DocC
    process.waitUntilExit()
  }

}
```

DocC: Objective-C and C support

Build tool plugins:

- plugins that can implement additional build steps
- similar to Xcode Rules
- example: source code generation, resource processing

Example:

```swift
import PackagePlugin

@main struct MyCoolPlugin: BuildToolPlugin {
  func createBuildCommands(context: TargetBuildContext) throws -> [Command] {

    let generatedSources = context.pluginWorkDirectory.appending("GeneratedSources")

    return [
      .buildCommand(
        displayName: "Running MyTool",
        executable: try context.tool(named: "mycooltool").path,
        arguments: ["create"],
        outputFilesDirectory: generatedSources)
    ]
  }
}
```

Plugins are declared in a new <kbd>Plugins</kbd> folder in the root of our Package, plugins are treated as Swift executables.

### Module aliasing

- solves module collisions (when two separate packages define a module with the same name)
- allows you to rename modules from outside the packages that define them

```swift
let package = Package(
  name: "MyStunningApp",
  dependencies: [
    .package(url: "https://.../swift-metrics.git"),
    .package(url: "https://.../swift-log.git") // ⚠️ swift-log and swift-metric define a Logging module
  ],
  targets: [
    .executableTarget(
      name: "MyStunningApp",
      dependencies: [
        .product(name: "Logging", package: "swift-log"),
        .product(name: "Metrics", package: "swift-metrics",
                 moduleAliases: ["Logging": "MetricsLogging"]), // 👈🏻 module aliasing
])])
```

## Performance improvements

- Built-time improvements
  - faster type-checking for generics

- Runtime improvements
  - optimized protocol conformance checking
    - protocol checking on app startup take as long as four seconds
    - from Swift 5.7, these checks are cached
    - for apps that rely heavily on Swift, this means launch-time can be up to twice as fast

### Swift driver

- program that coordinates the compilation of Swift source code in Swift
- rewritten in Swift (last year)
- can now be used as a framework directly inside the Xcode build system instead of as a separate executable
- this allows it to coordinate builds more closely with the build system to allow things like parallelization
- build times improvements around 5-25%

## Concurrency updates

- further fleshed out the model with data race safety at the forefront
- `distributed` actors
  - actors in different machines with a network between them
  - can write a safe backend for your app in Swift
  - note that a distributed actor call can potentially fail because of network errors
  - [new open source Distributed Actors package][swift-distributed-actors] that is focused on building server-side, clustered distributed systems in Swift 
    - integrated networking layer using SwiftNIO
    - implements the [SWIM consensus protocol][SWIM_Protocol] to manage state across the cluster

```swift
distributed actor Player {
   
  var ai: PlayerBotAI?
  var gameState: GameState
  
  distributed func makeMove() -> GameMove {
    return ai.decideNextMove(given: &gameState)
  }
}
```

- [Async Algorithms Package][aap]
  - seamless integration with `async/await`
  - home for time-based algorithms using `AsyncSequence`
  - support for Apple platforms, Linux, and Windows

- Actor prioritization
  - actors now execute the highest-priority work first

- new Swift concurrency instruments 
  - Swift Concurrency view helps you investigate performance issues
  - Swift Tasks and Swift Actors instruments provide a full suite of tools to help you visualize and optimize your concurrency code
    - provides useful statistics such as the number of tasks running simultaneously, total tasks that have been created up until that point in time
    - graphical representation of the parent-child relationships between tasks in structured concurrent code

## What's new in Swift 5.7

> Refer to [Apple's Swift repository CHANGELOG](https://github.com/apple/swift/blob/5fbede201a3e83d1957458060b6fbddf29494746/CHANGELOG.md#swift-57).

- `if let x = x` becomes `if let x` (optional unwrapping)
- No need to manually specify a closures result type (closure type inference)
- Permitted pointer conversion currently:

|   | Swift | C |
| --- | --- | --- |
| `UnsafePointer<UInt64> const uint64 t *` ↔️ `UnsafePointer<Int64> const int64 t*` | ❌ | ✅ |
| `UnsafePointer<mailmap_t> const mailmap_t *` ↔️ `UnsafePointer<UInt8> const char *` | ❌ | ✅ |
| `UnsafeRawPointer const void *` ↔️ `UnsafePointer<UInt8> const char *` | ❌ | ✅ |

- Pointer mismatches still possible when passing to C
- C++ working group works on improving the interoperation
- Special treatment for enabling conversions easily

|   | Swift | C | Swift calling C |
| --- | --- | --- | --- |
| `UnsafePointer<UInt64> const uint64 t *` ↔️ `UnsafePointer<Int64> const int64 t*` | ❌ | ✅ | ✅ |
| `UnsafePointer<mailmap_t> const mailmap_t *` ↔️ `UnsafePointer<UInt8> const char *` | ❌ | ✅ | ✅ |
| `UnsafeRawPointer const void *` ↔️ `UnsafePointer<UInt8> const char *` | ❌ | ✅ | ✅ |

- String parsing currently is hard (especially indices)
- “the problem is not the indices expression, it’s the whole function”
- Answer: Use Regexes!
- Literals supported now with compile-time checks

```swift
func parseLine(_ line: Substring) throws -> MailmapEntry {
  let regex = /\h*([^<#]+?)??\h*<([^>#]+)>\h*(?:#|\Z)/
  
  guard let match = line.prefixMatch(of: regex) else {
    throw MailmapError.badLine
  }
  
  return MailmapEntry(name: match.1, email: match.2)
}
```

- For those who find Regexes hard to read, new DSL:

```swift
import RegexBuilder

let regex = Regex {
  ZeroOrMore(.horizontalWhitespace)
  Optionally {
    Capture (OneOrMore(.noneOf("<#")))
  }
  .repetitionBehavior(.reluctant)

  ZeroOrMore(.horizontalWhitespace)
  "<"
  Capture (OneOrMore(.noneOf(">#")))
  ">"
  ZeroOrMore(.horizontalWhitespace)
  ChoiceOf {
    "＃"
    Anchor.endOfSubjectBeforeNewline
  }
}
```

- Requires `import RegexBuilder`
- Not a beginner-only feature
    - turn into reusable `RegexComponent`, use from other regexes, even are recursive
    - no special escaping needed
    - support literals inside of the builder
    - types can implement custom parsing logic, like `Date`

- `Regex` supports useful matching functions
- Custom engine written in Swift
- Compatible with UTS #18 unicode standard
- Available in macOS 13, iOS 16, tvOS 16, watchOS 9

### Generic code clarity 

Assume `Mailmap` is our generic:

| Something that conforms to Mailmap | A box whose contents conform to Mailmap |
| --- | --- |
| `struct EditableMailmap: Mailmap` | `var map: Mailmap` |
| `<Value: Mailmap>` | `Array<Mailmap>` |
| `where Element: Mailmap` | `where Element == Mailmap` |
| `some Mailmap` | `(Mailmap) -> Mailmap` |

We can be more expressive with any:


| Something that conforms to Mailmap | A box whose contents conform to Mailmap |
| --- | --- |
| `struct EditableMailmap: Mailmap` | `var map: any Mailmap` |
| `<Value: Mailmap>` | `Array<any Mailmap>` |
| `where Element: Mailmap` | `where Element == any Mailmap` |
| `some Mailmap` | `(any Mailmap) -> any Mailmap` |


- easier to explain the difference between methods and understand errors
- Swift will now “open the box” for us, so less “cannot conform to” errors
- Swift 5.7 fixes the “only be used as a generic constraint” error
- you can now specify the types in parameter, without the need to a `where` clause using “primary associated type”

```swift
protocol Collection<Element>: Sequence {
  associatedtype Index: Comparable
  associatedtype Iterator: IteratorProtocol<Element>
  associatedtype SubSequence: Collection<Element> where SubSequence.Index == Index, SubSequence.SubSequence == SubSequence

  associatedtype Element
}
```

- `AnyCollection` is a type-erasing wrapper, serves same purpose of `any`
- `any` is a built-in language feature that makes type-erasing wrappers no longer needed in many cases
- suggestion: replace your wrappers with typealiases, e.g. `let AnyCollection = any Collection` to keep backwards-compatibility (easier migration)
- Improvements to `any` types, but still have important limitations
- Don’t use `any` all the time, use generics instead, use the top one for better performance (generics)

```swift
func addEntries1<Entries: Collection<MailmapEntry>, Map: Mailmap>(_ entries: Entries, to mailmap: inout MailMap) {
  for entry in entries {
    mailmap.addEntry(entry)
  }
}

func addEntries2(_ entries: any Collection<MailmapEntry>, to mailmap: inout any Mailmap) {
  for entry in entries {
    mailmap.addEntry(entry)
  }
}
```

- `some ProtocolName` supported in more places, simplifies definition of Generics (simplifies above example, just write `some` instead of `any`)

```swift
func addEntries1(_ entries: some Collection<MailmapEntry>, to mailmap: inout some MailMap) {
  for entry in entries {
    mailmap.addEntry(entry)
  }
}

func addEntries2(_ entries: any Collection<MailmapEntry>, to mailmap: inout any Mailmap) {
  for entry in entries {
    mailmap.addEntry(entry)
  }
}
```

[swift-distributed-actors]: https://github.com/apple/swift-distributed-actors
[SWIM_Protocol]: https://en.wikipedia.org/wiki/SWIM_Protocol
[aap]: http://github.com/apple/swift-async-algorithms
[sketchnote]: https://fbernutz.github.io/images/sketchnotes/wwdc22-whats-new-in-swift.jpg
