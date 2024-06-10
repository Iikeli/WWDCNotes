# Create Swift Package plugins

Tailor your development workflow and learn how to write your own package plugins in Swift. We'll show you how you can extend Xcode’s functionality by using the PackagePlugin API to generate source code or automate release tasks and share best practices for creating great plugins.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110401", purpose: link, label: "Watch Video (24 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## What is a package plugin?

- Xcode runs plugins, plugins can communicate back
- Can be used for custom build tasks
- Also custom commands to SwiftPM CLI can be added

## Custom commands

- First, create a new folder in package called `Plugins`
- Create another nested folder for plugin target, e.g. `GenerateContributors`
- Create a Swift file inside, e.g. `plugin.swift`
- Make sure to bump the package tools version to 5.6 (or higher)
- Insert `.plugin` in the `targets` in your package manifest file:
    
```swift
.plugin(
  name: "GenerateContributors",
  capability: .command(
    intent: .custom(
      verb: "regenerate-contributors-list",
      description: "Generates the CONTRIBUTORS.txt file based on Git logs"
    ),
    permissions: [
      .writeToPackageDirectory(reason: "This command write the new CONTRIBUTORS.txt to the source root.")
    ]
  )
),
```

- `intent` can define a verb for the SwiftPM command line for `command` capability plugins
- `permissions` is used if you need to write into a directory (Sandbox)
- Implement by `import PackagePlugin` and conforming to `CommandPlugin` protocol
- To shell out to other tools, `import Foundation` and then call `Process().executableURL` and `.arguments`
- Define a `Pipe` to read output so you can use it in your logic

```swift
import PackagePlugin
import Foundation

@main
struct GenerateContributors: CommandPlugin {

  func performCommand(
    context: PluginContext,
    arguments: [String]
  ) async throws {
    let process = Process()
    process.executableURL = URL(fileURLWithPath: "/usr/bin/git")
    process.arguments = ["log", "--pretty=format:- %an <%ae>%n"]

    let outputPipe = Pipe()
    process.standardOutput = outputPipe
    try process.run()
    process.waitUntilExit()

    let outputData = outputPipe.fileHandleForReading.readDataToEndOfFile()
    let output = String(decoding: outputData, as: UTF8.self)

    let contributors = Set(output.components(separatedBy: CharacterSet.newlines)).sorted().filter { !$0.isEmpty }
    try contributors.joined(separator: "\n").write(toFile: "CONTRIBUTORS.txt", atomically: true, encoding: .utf8)
  }
}
```
    
- New command available in right-click context-menu of package
- Xcode will ask for permission showing the text in `permissions` from manifest file

## Plugins in detail

- Package plugins run in a Sandbox, access to work directory only
- To wrap another tool, make sure it only writes in work directory
- Build Tool Plugins structure (similar to run script phases in Xcode)
  - Executables
  - Inputs
  - Outputs

- Types of Build Tool Plugins
  - In-build command (if you have a clear set of outputs)
  - Pre-build command (if no clear outputs, but make sure to be performant!)

## In-build commands

- Plugins get run at the start of the build process
- Use `ProcessInfo().arguments` to read arguments in your own tool
- To create an executable, create new folder in `Sources`
    - Then add a `main.swift` file
    - Declare it in the `Package.manifest`

- Put the tool target into the `dependencies` of your plugin to use it
- Extend `BuildToolPlugin` instead of `CommandPlugin`

## Pre-build commands

- To make a plugin available to clients of a package, add it to `products` in `Package.swift` manifest file
