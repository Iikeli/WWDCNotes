# Creating Swift Packages

Whether you want to publish code to share with the community, or you just want a convenient way to organize the code in your apps, Swift packages are here to help. Learn how to create local packages for your own development, how to customize your package via the manifest file, and how to go about publishing a package for others to use.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/410", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Semantic Versioning

- Apple is really pushing us to use semantic versioning
- Use `x.x.x-alpha.1` `x.x.x-beta.2` for pre-releases

## Package Manifest File

- The core of a package is the Package Manifest File (which is always a `Package.swift` file in the project root):

```swift
// swift-tools-version:5.1 
import PackageDescription 

let package = Package(
	name: "MyPackage" 
)
```
￼
- The first line of the package declares the minimum version of the swift compiler required to build this package.
- We can use SPM with older versions of iOS, not only iOS 13 🎉

- The import declares that we’re going to use the swift package APIs in this file. This is a library provided by Xcode which contains the APIs used in the manifest file.

- Then we have a package initializer which is the real deal: it contains the package name, and more.

## Package Targets

- Targets are the basic building blocks of a package. A target can define a module or a test suite.

- Targets can depend on other targets in the same package, and on products in packages which this package depends on.

```swift
// swift-tools-version:5.1 
import PackageDescription 

let package = Package(
	name: "MyPackage", 
	targets: [ 
    .target(name: "MyTarget"),
  ]
)
```

- Each target has its own subfolder in the Sources folder, if we don’t like this default path, we can specify another path ourselves like this:
```swift
.target(name: "FloatingPanel", path: "Framework/Sources"),
```

- This path is crucial because Xcode will use it to read all the source files in there, it's basically where Xcode looks at when it wants to compile your package (we don’t need to list one by one all the files, but just tell Xcode this path).

- Test targets are declared in a similar way:

```swift
// swift-tools-version:5.1 
import PackageDescription 

let package = Package(
	name: "MyPackage", 
	targets: [ 
    .target(name: "MyTarget"),
    .testTarget( 
      name: "MyTargetTests", 
      dependencies:  ["MyTarget"]
    ),
  ]
)
```

- Note how we declare a dependency of one target to another.
- Test targets by default fall into a `Tests` folder instead of the `Sources` folder.

## Products

- Products define the executables and libraries produced by a package, and make them visible to other packages.
- Products are used to export target from a package so other packages can use them.

```swift
let package = Package(
	name: "MyPackage",
	products: [
		.library (
			name: "MyProduct",
			targets: ["MyTarget"]
		),
	],
	...
)
```

- By default all the packages are static, if we want to load them at run time, we need to specify their type to dynamic:

```swift
.library( 
  name: "LegacyMenuDownloader",
  type: .dynamic, 
  targets: ["LegacyMenuDownloader"]
),
```

- In case our package has dependencies, we will declare them like this:

```swift
let package = Package(
	name: "MenuDownloader",
	products: [...], 
  dependencies: [ 
    .package (url: "https://github.com/jpsim/Yams", .upToNextMajor(from: "2.0.0")),
  ], 
  targets: [...]
)
```
￼
- Beside versions, we can also declare branches and specific commits as well:

```swift
// Branch-based requirement. 
.package(url: "https://github.com/jpsim/Yams", .branch("main"))

// Revision-based requirement. 
package(ur1: "https://github.com/jpsim/Yams", revision("85cfe06"))
```

- Note that these two last requirements (branch/revision)  are not allowed in public packages, we must use the version requirement instead.

## Platform Availability

- Minimum versions that support SPM:
![][osImage]

- We can declare in our package the version required by our package:

```swift
let package = Package(
	name: "MyPackage",
	platforms: [
	.macOS(.v10_15), .iOS(.v13)
	],
	...
)
```

- Note how this does not restrict the usage of the package only on those platforms: this parameter tells which platform minimum version is required for that specific version in order to run.

- Package dependencies are locked from editing (you can’t edit your dependencies): however, once you have a local copy of the dependency in your project, that copy overrides the remote dependency and you can edit it.

## Limits

- Currently SPM supports source code and unit tests, no images or other resources.

[osImage]: WWDC19-410-os