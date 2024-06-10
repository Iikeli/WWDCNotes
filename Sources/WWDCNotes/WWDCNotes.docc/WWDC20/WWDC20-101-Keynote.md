# Keynote

The Apple Worldwide Developers Conference kicks off with exciting reveals, inspiration, and new opportunities to continue creating the most innovative apps in the world. Join the worldwide developer community for an in-depth look at the future of Apple platforms, directly from Apple Park.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/101", purpose: link, label: "Watch Video (108 min)")

   @Contributors {
      @GitHubUser(abadikaka)
      @GitHubUser(zntfdr)
   }
}



## iOS 14

![][iosImage]

- **App Library**: A new space for organizing all of our apps into a grouped library and easy to navigate. We can hide unused screens full of icons.
- **Widgets**: Introducing widgets in the home screen, alongside app icons. We’re moving away from translucent widgets, we have colors that pop now. Widgets can be of three different sizes, that takes a different number of icons space. Today view remains.
  - **Smart Stack**: The smart stack is a special widget that varies throughout the day (based on our device usage): we might see news in the morning, calendar at noon, and fun activities in the evening.

- **Picture in picture**: PiP is now on iOS as well.
- **Siri**: Redesigned Siri experience, less intrusive and blocking. Siri is about 20x faster and her knowledge has widen.
- **Translate.app**: Brand new translation app with advanced on-device machine learning and a powerful neural engine. With this app, Apple has shelocked 2018 Apple Design Winner [iTranslate Converse][iTranslateApp].
- **Messages.app**: Pinned conversations, inline replies, mentions, new animations, and more memoji.
- **Maps**: 
  - **Cycling directions**
  - **Guides**: A collection of always up-to-date guides on great places to eat, explore, etc.
  - **EV Routing**: Navigations will track all of our current charges, elevation, weather, congestion and green zone to easily where they are along with routing options.

- **CarKey**: Lets use our devices as the key of the car (if our car is compatible), lets us share our key to other people via temporary sharing.
- **Digital CarKey**: Leave our key at home and use digital key to start our engine (support only BMW 104 i for now)
- **App Clips**: use apps on the go, without installing the whole app, 10MB max, similar to android instant apps.

## iPadOS 14

![][ipadosImage]

- **Sidebar**: It's a new powerful UI element that enables new capabilities and features such as drag-drop, switching tabs and more.
- **Compact design**: Siri, incoming calls, and search have a new, less intrusive and blocking design.
- **Scribble**: Handwrite into any text field and automatically convert to text, works with the pencil.

## Airpod
- **Automatic Switching**: Seamlessly move between devices without manually switching it.
- **Spatial Audio**: Play sound virtually everywhere and creating an immersive sound experience, AirPods Pro only.

## watchOS 7

![][watchosImage]

- **Watch Face Sharing**: Easily get and share watch faces, watch faces can be downloaded from the web, or donated by apps.
- **Dance Tracking**
- **Sleep Tracking**
- **Wind Down**: Minimizes distraction and helps us maintaining a good routine
- **Hand Wash Detection**: Automatic detection when washing hand, the watch will start a 20 seconds animation to let you know when you've adequately washed your hands.
- **Fitness**: the Activity app has a new name

## Location, Privacy and Security
- **Sign In With Apple**: lets developers offer an option to migrate our account to Sign In With Apple.
- Option to only share only approx. location.
- Camera/microphone indicator on the status bar when they're turned on.
- Privacy displayed in the app store store, no need to download an app to know its policies, developer need to self-report this.

## Home

![][homeImage]

## macOS 11

![][macosImage]

- macOS 11, Big Sur.
- Mostly a design overhaul, macOS has never looked so close to iOS: more rounded corners, more translucency
- App icons look like iOS/iPadOS icons (squircles), with more skeuomorphism
- Sidebar, like in iPadOS, is a new powerful UI component.
- Mail and Photos app redesign
- New control center, and notification center, identical to iPadOS. Quick settings is a new shortcut to system settings like brightnes and light/dark mode toggle, they seem to be designed for touch.
- **Messages** and **Maps** are now catalyst apps, gaining the same new feature seen in iOS/iPadOS.
- Catalyst has gotten many new features:
![][catalystImage]
- Safari:
  - In place translations.
  - Enhancement on Safari extensions

## ARM (Apple Silicon)

- The full mac line up will run on arm in about two years
- There still are more intel devices coming down the pipeline
- The first arm devices will be released by end of the year
- **Universal 2**: build an app for both intel and arm targets.
- **Rosetta 2**: run intel apps on arm (translated at installation, transcoded on the fly for [JIT](https://en.wikipedia.org/wiki/Just-in-time_compilation))
- **Virtualization**: mac users will able to run Linux and Docker on an arm device
- **iOS Apps**: mac users will be able to run iOS and iPadOS apps on an arm (mac) device
- **Developer Transition Kit**: to start working on bringing support to ARM mac, Apple is offering a development kit, which is a mac mini running on A12Z Soc, 16GB memory,  512GB SSD.

[iTranslateApp]: https://apps.apple.com/us/app/itranslate-converse/id1241264761

[iosImage]: ios.jpeg
[ipadosImage]: ipados.jpg
[watchosImage]: watchos.png
[homeImage]: home.jpg
[macosImage]: macos.jpg
[catalystImage]: catalyst.jpeg