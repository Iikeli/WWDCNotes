# Meet Swift Package plugins

Discover how you can perform actions on Swift packages and Xcode projects with Swift package plugins. We'll go over how these plugins work and explore how you can use them to generate source code and automate your development workflow.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110359", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



## What is a package plugin?

- Swift script that can be run as part of your build
- A package could have plugins as extra, or be all about plugins
- Package plugins are available only within package
- General plugins can be made available to the outside
- Lets you access development tools on your machine

### Command plugins:

- implement custom actions, such as formatters, linters, prepare distribution
- can ask for permission to modify files in package
- some might just read to report some data

### Build tool plugins:

- extend the dependency graph
- applied to each target that needs them

## Demo

- add plugin like any other package dependency in project, via the `dependencies` in your package manifest file:
    
```swift
let package = Package (
  name: "CoreLib",
  products: [
    .library(name:"CoreLib", targets: ["CoreLib"])
  ],
  dependencies: [
    // 👇🏻
    .package (url: "git@example.com:DemoPackages/SourceCodeUtilityPlugins.git", branch: "main")
  ],
  targets: [
    .target (name: "CoreLib"),
    .testTarget(name: "CoreLibTests", dependencies: ["CoreLib"])
  ]
)
```

- the plugin will be available once downloaded (no need to add it as a dependency on target or similar)
- When running plugins, you choose the target and can pass extra arguments
- Xcode asks permission for plugins that modify your files, has a <kbd>show code</kbd> button

## How do plugins work?

- plugins can create files, read dependencies
- plugins run in a Sandbox
- plugin can send results back to Xcode, such as warnings and errors
- use `import PackagePlugin` to implement one
    
```swift
import PackagePlugin

@main
struct MyPlugin: ... {

    // Entry points specific to plugin capability. These entry points are invoked
    // when the plugin is applied to a package.

}

// 👇🏻 use #if canImport for conditional support for Xcode projects
#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: ... {

  // Entry points specific to plugin capability. These entry points are invoked
  // when the plugin is applied to an Xcode project.

}
#endif
```

Two plugin types:

- Command plugins
- Build tool plugins

## Command plugins

- implement development-time actions
- Run interactively, not during a build (e.g., manually triggered by the developer)
- Must ask for permission to modify package sources
- Might depend on other tools to do the actual work
- dependencies on tools plugins can be both binaries or source code
- run `swift package plugin --list` to see which plugins are available
- run `swift package <plugin_name>` to run them in current directory
- run with `swift package --allow-writing-to-package-directory <plugin_name>` to give write access
- append `--verbose` to see more output from plugin

## Build tool plugins

- provide commands for the build system, triggered at build time (not manually)
- can invoke executable provided as binaries or built from source
- supports build commands and prebuild commands
- output files are stored with other build artifacts, not among your package source code
- commands run in a sandbox to prevent changes in package
- Requires conforming to `BuildToolPlugin` protocol, return type is `[Command]`
    
```swift
import PackagePlugin

@main
struct MyPlugin: BuildToolPlugin {

  /// This entry point is called when operating on a Swift package.
  func createBuildCommands(context: PluginContext, target: Target) throws -> [Command]
    debugPrint(context)
    return []
  }
}

// 👇🏻 use #if canImport for conditional support for Xcode projects
#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: XcodeBuildToolPlugin {

  /// This entry point is called when operating on an Xcode project.
  func createBuildCommands(context: XcodePluginContext, target: XcodeTarget) throws -> [Command]
    debugPrint(context)
    return []
  }
}
#endif
```
    
- Build commands run as part of the build
  - You specify input and output paths
  - They run only when their output is missing, or when their inputs (e.g. files, assets) has changed (they're skipped otherwise)

- Prebuild commands run before the build starts
- Add plugins to targets via `plugins` parameter
    
```swift
let package = Package (
  name: "CoreLib",
  products: [
    .library(name:"CoreLib", targets: ["CoreLib"])
  ],
  dependencies: [
    // 👇🏻
    .package (url: "git@example.com:DemoPackages/MySourceGenPlugin.git", from: "1.0.0")
  ],
  targets: [
    .target (
      name: "CoreLib",
      plugins: [
        .plugin(name: "MySourceGenBuildTool", package: "MySourceGenPlugin") // 👈🏻 required!
      ]
    ),
    .testTarget(name: "CoreLibTests", dependencies: ["CoreLib"])
  ]
)
```
