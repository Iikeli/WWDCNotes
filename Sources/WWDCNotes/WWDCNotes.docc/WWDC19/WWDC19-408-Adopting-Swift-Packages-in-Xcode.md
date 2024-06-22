# Adopting Swift Packages in Xcode

Swift packages are a great way to organize and share code, and are now supported while building apps for all Apple platforms in Xcode 11. Find out how to use community-developed packages in your project, how Swift packages are structured, and how package versioning and dependencies work.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/408", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Swift Package Manager

- We now have a built-in package manager.
- We can even search for new packages directly in Xcode (we might need to log in with GitHub first).
![][Image]
- Packages can contain Swift as well as C, Objective-C.
- When adding a package in a project, a new `swiftpm` folder is created under `xcshareddata`, this should be committed as it contains the resolved versions of our dependencies (stored in `Package.resolved` file).
- A `Package.swift` file indicates that this folder is a package.
- What Packages enable us is to import new, potentially 3rd party modules in our project
- Underneath the sources is a subdirectory for each of the separate targets in the package. These are the separately buildable components of the package. And similarly, under the test directory, there's a separate subdirectory for every test suite. 
- The products section in the manifest lists the products that the package vends to the clients. The Package can control which parts of its code can be directly imported by the client. 
- `.library(name: “myName”, targets: “Yams”)` basically says that this library publishes the `Yams` target to clients as a library. 
- The target section lists the individually buildable parts of the package. 
- A Package module/product is build for each platform you need (iOS, watchOS, ..), this is why you need to import the package in the project editor for each target.
- Package libraries are **static by default**.

## Package resolution conflicts

- In case of multiple dependencies, we can have only **up to one version of a package per workspace**.
- If one of my dependency relies on a package version 2.x.x, and I want to use a version 3.x.x, that won’t work. either the dependency upgrades to 3.x.x, or I downgrade to 2.x.x


[Image]: WWDC19-408-menu