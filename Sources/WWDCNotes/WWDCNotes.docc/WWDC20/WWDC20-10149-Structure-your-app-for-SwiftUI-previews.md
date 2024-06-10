# Structure your app for SwiftUI previews

When you use SwiftUI previews during development, you can quickly create apps that are more flexible and maintainable. Discover ways to improve the preview experience by making small tweaks to your project. Find out how to preview multiple files at once, how to manage data flow for previews, and how to use sample data while previewing. We'll also give you strategies for defining view inputs to make them more previewable and testable.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10149", purpose: link, label: "Watch Video (33 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Structure your app for SwiftUI previews

## Preview Multiple Files At Once

- Pin previews by tapping the pin symbol in the preview canvas bottom bar. 
- Only one preview can be pinned.
- By pinning a preview we can browse our project, edit files etc, and those previews will still be displayed on the screen.

## App Life Cycle

- It's important to define explicit dependencies for our data.
- Models that are created using `@StateObject` are not initialized in previews.

## Where to define sample data 

In the `General` tab of our project editor we have a `Development Assets` section: 

- it contains paths to files and folders that Xcode include only in the `development` configuration of our app
- we can add both assets and code (for preview sample data for example)

## Structure Views To Be Previewable

For different ideas on how to do so

### 1. Immutable Inputs

- Pass only basic types such as `Int` and `String`
- Pass down to the view only data that the `View` needs.
- For buttons actions and similar, pass a closure.

### 2. Mutable Inputs (`@Binding`)

- Use `.constant()` for fixed `@Binding` previews
- For dynamic previews:
  - create an interemediate `View`, that behaves as the `@State` container
  - the `body` of this container will be the view that we would like to preview, injected with the container `@State`

### 3. Generics Inputs

Instead of passing actual explict types instances to a `View`s: 

- we can make our `View`s generic
- we can pass instances of a protocol that conforms to `ObservableObject`, `Identifiable` and that provides anything that our `View` needs

4. Use Environment

We can pass environment objects directly in the preview

## Xcode Preview Toolbar

- In Xcode 12 above each preview we have a new toolbar.
- This toolbar displays the current state of the preview and the title in the middle and two buttons on both sides:
  1. the leading button is used to either start the live preview or to attach the debugger to the preview.
  2. the next button is used to send the preview to a real connected device
    - previewing on device will also display a `Previews` app on the homescreen, which will open the last preview used in that phone, even when the phone is disconnected from the Xcode

- 
  3. the second trailing button lets set particular environment states and add padding to the preview
  4. the last button creates a duplicate of the preview
