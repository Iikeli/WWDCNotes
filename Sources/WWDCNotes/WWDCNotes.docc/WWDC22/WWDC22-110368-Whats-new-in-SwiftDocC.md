# What's new in Swift-DocC

Join us for an exciting update on Swift-DocC and learn how you can write and share documentation for your own projects. We'll explore improvements to Swift-DocC navigation and share how you can compile documentation for application targets and Objective-C code. We'll also show you how to publish your content straight to hosting services like GitHub Pages.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110368", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(dagronf)
   }
}



## Open source

* Is now an open source project. 
* Current release is a collaboration with community.

## App projects

### Documentation catalogs

A 'bundle' of documentation. Contains additional markdown files and media

1. Right click on your project's source folder
2. Select 'New File...'
3. Select 'Documentation Catalog'

Xcode automatically creates a top level document for you to fill in.

Can be linked to in your code via Xcode-docc link format eg

```swift
/// ``MyApp``  (MyApp is the name of documentation catalog)
```

**Note:** With Xcode 14 beta 1 it appears that you can only have a single documentation catalog. Creating more than one results in a compile error. Unknown if this is a bug or a limitation.

Good approach for app project documentation :-

1. Start with each individual API
2. Build higher level content and concepts using a documentation catalog.

## Language support

* Now supports Swift, Objective-C and C.
* Use the same documentation markup for swift, objective-c and c code.
* Generated documentation for code that can be accessed by multiple languages will contain a language toggle.

## Publish

* DocC archive (`.doccarchive`) has always contained a fully featured website representation.
* With Xcode14 the `.doccarchive` is directly compatible with most managed hosting services (eg. GitHub pages)
* In most cases, you can deploy your documentation by copying the ***contents*** of your built `.doccarchive` into the root of your web server.

**If you cannot deploy at the root path of your url (eg. github pages)**

* set a **base path** in the projects build settings `DocC Archive Hosting Base Path`

eg. github pages :-

`https://username.github.io/<your repository name>/documentation/<projectname>`

The `DocC Archive Hosting Base Path` should be set to `<your repository name>`

eg.

Hosting location is at `https://username.github.io/sloth-creator/documentation/slothcreator/`

The `DocC Archive Hosting Base Path` should be set to `sloth-creator`

Then whenever the docc archive is re-built the ***base path*** for the html content will be at `sloth-creator`

Copy the content of the `.doccarchive` to the `sloth-creator` folder.

* [More online hosting info](https://apple.github.io/swift-docc-plugin/documentation/swiftdoccplugin/generating-documentation-for-hosting-online)
* [Publishing to GitHub pages](https://apple.github.io/swift-docc-plugin/documentation/swiftdoccplugin/publishing-to-github-pages)

## Swift Package Manager integration

[Can be found here](https://apple.github.io/swift-docc-plugin/documentation/swiftdoccplugin/)

* Swift 5.6 or higher is required in order to run the plugin.
* can be used to re-deploy the documentation any time the documentation changes.
* Build documentation for the package and dependencies using `swift package generate-documentation`

Requires a new dependency in your Package.swift file

```swift
let package = Package(
    // name, platforms, products, etc.
    dependencies: [
        // other dependencies
        .package(url: "https://github.com/apple/swift-docc-plugin", from: "1.0.0"),
    ],
    targets: [
        // targets
    ]
)
```

## Navigation sidebar

Generated web documentation now has a navigation sidebar

Has a 'filter' field at the bottom of the navigation.
