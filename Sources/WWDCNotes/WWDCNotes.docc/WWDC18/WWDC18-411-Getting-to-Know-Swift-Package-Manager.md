# Getting to Know Swift Package Manager

The Swift Package Manager makes it possible to easily develop and distribute source code in the Swift ecosystem. Learn about its goals, design, unique features, and the opportunities it has for continued evolution.

@Metadata {
   @TitleHeading("WWDC18")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc18/411", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



The Swift Package Manager makes it easier to develop and distribute source code in the Swift ecosystem.

## Why a package manager in Swift?

- great cross-platform tool for building your Swift code.
- makes it easy to configure your code in a consistent way and run it on all of Swift's supported platforms.
- includes its own complete build system, allowing you to configure your software, build it, test it, and even run it from one tool.
- new standard for distributing libraries.

## How to use it

SwiftPM consists of four command tools at the top level `swift` Command:

- `$ swift build` to build your package
- `$ swift run` to run your package executable products
- `$ swift test` to run your package tests
- `$ swift package` (to run various non-build operations on the package)

### Create a package

Use `$ swift package init` to create a new library package, add `--type executable` to create an executable package.

This command creates the package in the current directory, the package will be composed with:

- a `Package.swift` manifest file, which describes the structure of the package.
- a basic `README.md`
- the `Sources` directory with a subfolder for our package target
- the `Tests` directory where we can add unit tests

## The anatomy of a package

A package is composed by three main parts:

- dependencies
- targets
- products

### Dependencies

- Swift packages that you can use when developing your package features.
- Each dependency provides one or more products such as libraries that your package can use.
- Each dependency has a source location and it is versioned.

### Targets

- Basic building blocks of packages.
- Describes how to build a set of source files into either a module or a test suite.
- Targets can depend on other targets of the same package and on products exported from other packages, declared as dependencies.
- A target can contain any C language (C, C++, ObjC) or Swift, both language families are allowed, but not in the same target (they must be separate).

### Products

- Products are executable to libraries and products are assembled from the build artifacts of one or more target.
- Packages provide libraries for other packages by defining products.

## The design of SwiftPM

SwiftPM follows Swift's philosophy:

- Safe: isolated build environment
- Fast: scalable to large dependency graphs
- Expressive: Swift language manifest format

### Building a package

- SwiftPM uses [`llbuild`][llbuild] to build a package: `llbuild` is a set of libraries for building build systems, it's built around a general purpose and reusable build engine.
- SwiftPM builds packages in isolation: ensures that even packages with complex requirements can be reliably built and used in different environments.
- builds are sandboxed: nothing can write to arbitrary locations on the file system during the build.
- No arbitrary commands or shell scripts during a build: This allows SwiftPM to fully understand any package build graph and all of its inputs and outputs to do fast, correct incremental builds.

### Workflow features

- Edit Mode: allows overwriting a specific package with a local copy, so that temporary edits can be made, and changes to transitive dependencies can be tested without having to forward all packages in the graph upfront.
- Branch dependencies: (works only on development mode) allows your package to have dependencies targeting a branch instead of using the dependency versioning

[llbuild]: https://github.com/apple/swift-llbuild
