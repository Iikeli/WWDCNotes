# Swift packages: Resources and localization

Bring your resources along for the ride when you organize and share code using Swift packages. Discover how to include assets like images and storyboards in a package and how to access them from code. And learn how to add localized strings to make your code accessible to people around the world.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10169", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Adding Resources to a Package

- New with Xcode 12/Swift 5.3
- Adding resources to a package works with existing APIs
- Adding resources does not affect the OS version requirements of our package
- To add resoruces, we create an assets catalog, a.k.a. a folder with `.xcassets` suffix, the same way as an Xcode project.
- to access to the package resources, we need to pass a bundle parameter where the resource is needed, for example: `Image("myPackageImage", bundle: .module)` 

### Resource purpose

- Some resources (as seen in the image below) are automatically added in the package assets, for example:

![][clearResourcesImage]

- Some other resources, that are not places in an assets catalog (for development purposes for example), need to have an intent declared in the package manifest

![][ambiguousResourcesImage]

- In order to exclude assets or folders to be embed in our package, add these resources in the `excluded` array of the target
- In order to include standalone assets (a.k.a. assets not in an asset catalog), we need to declare them in with the `resources` array of its target.
  - most resources should use the `.process` action, so that they are transformed as appropriate at build time, the exact processing that depends on the platform for which the package is built.
  - in case we would like to have a full folder available at runtime, while also preserving its structure as it is, we will use the `.copy` type instead

```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(name: "MyPackage",
    products: [
        ...
    ],
    targets: [
        .target(name: "MyLibrary",
            excludes: [
                "Internal Notes.txt", // excluding this file
                "Artwork Creation"], // excluding this folder content on build time
            resources: [
                .process("Logo.png"), // processed and available at runtime
                .copy("Game Data")] // copied and available at runtime exactly as it is
        )
    ]
)
```

## Localizing Resources

- first we need to add a default localization in the package declaration

```swift
let package = Package(
    name: ...
    defaultLocalization: "en", // mandatory if our package supports localizations
    products: [
        ...
    ],
    targets: [
        ...
    ]
)
```

- it works like an Xcode project: we create the languages `en.lproj` folders at the root of our target and add a `Localizable.strings` file in each with the localizations

- like for assets, wherever our localizations are needed we need to specify the bundle, for example: `Text("user_greeting", bundle: .module)`

- no need to declare the `.lproj` in the package manifest
- localized resources within the assets catalog are also supported

[clearResourcesImage]: WWDC20-10169-clearResources
[ambiguousResourcesImage]: WWDC20-10169-ambiguousResources
