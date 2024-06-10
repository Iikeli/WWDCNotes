# What’s new in SwiftUI

Learn how you can use SwiftUI to build great apps for all Apple platforms. Explore the latest updates to SwiftUI and discover new scene types for visionOS. Simplify your data models with the latest data flow options and learn about the Inspector view. We’ll also take you through enhanced animation APIs, powerful ScrollView improvements, and a host of refinements to help you make tidier tables, improve focus and keyboard input, and so much more.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10148", purpose: link, label: "Watch Video (34 min)")

   @Contributors {
      @GitHubUser(DaemonLoki)
      @GitHubUser(MortenGregersen)
      @GitHubUser(multitudes)
   }
}



## Chapters
[1:05 - SwiftUI in more places](https://developer.apple.com/videos/play/wwdc2023/10148/?time=65)  
[10:21 - Simplified data flow](https://developer.apple.com/videos/play/wwdc2023/10148/?time=621)  
[18:46 - Extraordinary animations](https://developer.apple.com/videos/play/wwdc2023/10148/?time=1126)  
[27:18 - Enhanced interactions](https://developer.apple.com/videos/play/wwdc2023/10148/?time=1638)  

## Intro
- SwiftUI in more places  
- Simplified data flow  
- Extraordinary animations  
- Enhanced interactions  

# SwiftUI in more places
Spatial computing brings SwiftUI into a bold new future with all-new 3D capabilities like volumes; rich experiences with immersive spaces; new 3D gestures, effects, and layout; and deep integration with RealityKit. From core pieces like the Home View in Control Center to familiar apps like TV, Safari, and Freeform; to all-new environments like immersive rehearsals in Keynote; SwiftUI is at the heart of these user experiences

![Overview of the section titled 'SwiftUI in more places'][1]

[1]: WWDC23-10148-SwiftUI-in-more-places

### Scenes for spatial computing

Scenes for spatial computing uses `WindowGroup` (render as 2D windows with 3D controls).
Add `.windowStyle(.volumetric)` to get a volume.

![WindowGroup][WindowGroup]

[WindowGroup]: WWDC23-10148-WindowGroup

### NavigationSplitViews are available.

```swift
NavigationSplitView {
    SidebarList($selection)
} detail: {
    Detail(for: selection)
}
```

![NavigationSplitViews][NavigationSplitViews]

[NavigationSplitViews]: WWDC23-10148-NavigationSplitViews


### TabViews

```swift
TabView {
    DogsTab ()
        .tabItem { ... }
    CatsTab()
        .tabItem { ... }
    BirdsTab ()
        .tabItem { ... }
}
```

![TabViews][TabViews]

[TabViews]: WWDC23-10148-TabViews

### Volume content

Fill a volume with a static model using `Model3D`. 

![volume content][volume]

[volume]: WWDC23-10148-globe

For dynamic, interactive models with lighting effects and more, use the new `RealityView`.
```swift
// Volume content
import SwiftUI 
import RealityKit

struct Globe: View {
    var body: some View {
        RealityView { content in
            if let earth = try? await ModelEntity(named: "Earth") {
                earth.addImageBasedLighting()
                content.add(earth)
            }
        }
    }
}
```
### Immersive Spaces
Use an ImmersiveSpace with the mixed immersion style to connect your app to the real world, combining your content with people's surroundings. Anchor elements of your app to tables and surfaces, and augment and enrich the real world with virtual objects and effects. Go further with the full immersion style. Your app takes complete control. Build these connected and immersive experiences using the same Model3D and RealityView that work in volumes.
- full immersion style
- mixed immersion style

See more in the session:

[Meet SwiftUI for spatial computing - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10109)  

## watchOS 10 redesigned user experience

![watchOS 10 redesigned user experience][watch]

[watch]: WWDC23-10148-watch

- Newly empowered with new transitions:
    - NavigationSplitView
    - TabView gets a new `.verticalPagingStyle` driven by the Digital Crown
    - NavigationStack

![watchOS 10 redesigned user experience][watch2]

[watch2]: WWDC23-10148-watch2

- `.containerBackground` modifier lets you configure the background of the container which animate when you push and pop content.

```swift
// Container backgrounds
CityDetails(city: city)
    .containerBackground(for: navigation) { ... }
```

![watchOS 10 redesigned user experience][watch3]

[watch3]: WWDC23-10148-watch3

- Multiplatform toolbar placements (`.topBarLeading` and `.topBarTrailing`, along with the existing `.bottomBar`) let you perfectly place small detail views in your Apple Watch apps.

```swift
// Toolbar placements
.toolbar {
    ToolbarItem(placement: .topBarTrailing) { ... }
    ToolbarItem(placement: .bottomBar) { ...}
}
```

![watchOS 10 redesigned user experience][watch4]

[watch4]: WWDC23-10148-watch4

- New additions
    - `DatePicker`
    - Selection in `List`s

```swift
// Newly available
DatePicker(
    "Time (h:m:s)",
    selection: $date,
    displayedComponents: hourMinuteAndSecond
)

NavigationSplitView {
    List(locales, selection: $selectedLocale) { locale in
        ...
    } 
    .containerBackground(...)
} detail: {
    if let selectedLocale { ... }
}
```

![watchOS 10 redesigned user experience][watch5]

[watch5]: WWDC23-10148-watch5


See more in the sessions:
[Design and build apps for watchOS 10 - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10138)  
and  
[Update your app for watchOS 10](https://developer.apple.com/videos/play/wwdc2023/10031)  

## Interactive widgets 
Widgets for the Smart Stack on watchOS 10 let the people using your app see their information on the go.

![watchOS 10 redesigned user experience][watch6]

[watch6]: WWDC23-10148-watch6

SwiftUI is core to widgets wherever they appear, like these other new places. Widgets on the Lock Screen on iPadOS 17 are a great complement to widgets on the Home Screen.

![iPad Widgets][iPadWidgets]

[iPadWidgets]: WWDC23-10148-iPadWidgets


Big, bold widgets shine on the iPhone Always-On display with Standby Mode. 


And desktop widgets on macOS Sonoma...

## Interactivity and interaction
Widgets now support interactive controls. Toggle and Button in Widgets can now activate code defined in your own app bundle using App Intents. And you can animate your widgets using SwiftUI transition and animation modifiers.

- Widgets for Smart Stack
    - Appear when scrolling down
    - Also available on iPadOS 17
    - Standby Mode for iPhone
    - Desktop Widgets on macOS
- Interactive controls for Widgets
    - Toggle and Button
    - Ase App Intents
    - animate with transitions and animations
    - Previews leverage macros to allow you to show the widget states with a timeline

See more in the sessions:

[Bring widgets to new places - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10027) 

and 

[Bring widgets to life - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10028/)  

## Previews for widgets
```swift
#Preview(as: .systemSmall) {
    CaffeineTrackerWidget()
} timeline: {
    CaffeineLogEntry.log1
    CaffeineLogEntry.log2
    CaffeineLogEntry.log3
    CaffeineLogEntry.log4
}
```

#### Xcode Previews
Declare and configure a Preview, add a widget type, and define a timeline for testing. Xcode Previews shows the current widget state and a timeline that lets you see the animations between states. Of course, the new previews work with regular SwiftUI views and apps as well.


```swift
import SwiftUI 
import AppIntents

struct CaffeineTrackerwidgetView : View {
    var entry: CaffeineLogEntry
    
    var body: some View {
        VStack (alignment: leading) {
            TotalCaffeineView(totalCaffeine:
                entry.totalCaffeine)
                
        Spacer ()
        
        if let log = entry.log {
            LastDrinkView(log: log)
        }
        
        Spacer ()
        
        HStack {
            Spacer ()
            LoaDrinkView()
            Spacer ()
            }
    }
    .fontDesign (.rounded)
    .containerBackground(for: .widget) {
        Color.cosmicLatte
        }
    }
}
```

![Widgets][widgets]

[widgets]: WWDC23-10148-widgets

New Previews syntax:

```swift
#Preview("good dog") {
    ZStack(alignment: .bottom) {
        Rectangle()
            .fill(Color.blue.gradient)
        Text("Riley")
            .font(.largeTitle)
            .padding()
            .background(.thinMaterial, in: .capsule)
            .padding()
    }
    .ignoresSafeArea()
}
```

You can now interact with previews of Mac apps right inside Xcode.

See more in the sessions:

[Build programmatic UI with Xcode Previews - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10252)

and

[What’s new in Swift - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10164)  

### SwiftUI extensions to other frameworks
Several frameworks bring new or improved support:  
- Authentication Services / MapKit
- HealthKit Swift Charts Continuity device picker
- Apple Pay Later / StoreKit / In-app purchasing / CareKit

### MapKit
MapKit delivers a massive update:
```swift
// MapKit

import SwiftUI 
import MapKit

Map {
    Marker(item: destination)
    MapPolyline(route)
        .stroke(.blue, lineWidth: 5)    
    UserAnnotation()
}
.mapControls {
    MapUserLocationButton( )
    MapCompass()
    }
```

![mapkit][mapkit]

[mapkit]: WWDC23-10148-mapkit


**NEW:** Use maps in your view.
**NEW:** Add markers, polylines and the user's location.

```swift
import SwiftUI
import MapKit

struct Maps_Snippet: View {
    private let location = CLLocationCoordinate2D(
        latitude: CLLocationDegrees(floatLiteral: 37.3353),
        longitude: CLLocationDegrees(floatLiteral: -122.0097))

    var body: some View {
        Map {
            Marker("Pond", coordinate: location)
            UserAnnotation()
        }
        .mapControls {
            MapUserLocationButton()
            MapCompass()
        }
    }
}

#Preview {
    Maps_Snippet()
}
```

See more in the session:

[Meet MapKit for SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10043)   


### SwiftCharts


```swift
// Swift Charts
Chart {
    ForEach(SalesData.last365Days, id: \.day) {
        BarMark(
            x: .value("Day", SO.day, unit: .day),
            y: .value("Sales", $0.sales)
        )
    } 
    .foregroundStyle(.blue)
} 
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 3600 * 24 * 30)
.chartScrollPosition(x: $scrollPosition)
```

![charts][charts]

[charts]: WWDC23-10148-charts

**NEW:** Scrolling and selection in charts.

```swift
import SwiftUI
import Charts

struct ScrollingChart_Snippet: View {
    @State private var scrollPosition = SalesData.last365Days.first!
    @State private var selection: SalesData?

    var body: some View {
        Chart {
            ...
        }
        .chartScrollableAxes(.horizontal)
        .chartXVisibleDomain(length: 3600 * 24 * 30)
        .chartScrollPosition(x: $scrollPosition)
        .chartXSelection(value: $selection)
    }
}
```

![charts][charts2]

[charts2]: WWDC23-10148-charts2


**NEW:** Donut and pie charts with the new `SectorMark`.

```swift
import SwiftUI
import Charts

struct DonutChart_Snippet: View {
    var sales = Bagel.salesData

    var body: some View {
        Chart(sales, id: \.name) { element in
            SectorMark(
                angle: .value("Sales", element.sales),
                innerRadius: .ratio(0.6),
                angularInset: 1.5)
            .cornerRadius(5)
            .foregroundStyle(by: .value("Name", element.name))
        }
    }
}
```

![charts][charts3]

[charts3]: WWDC23-10148-charts3


See more in the session:

[Explore pie charts and interactivity in Swift Charts - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10037)  

### StoreKit



**NEW**: Cross platform view, for presenting available subscriptions. The view can be customized.


```swift
// StoreKit
import SwiftUI 
import StoreKit

SubscriptionStoreView(groupID: passGroupID) {
    PassMarketingContent()
        .lightMarketingContentStyle()
        .containerBackground(
            for: .subscriptionStoreFullHeight
            ) {
            SkyBackground()
            }
} 
.backgroundStyle(.clear)
.subscriptionStoreButtonLabel(.multiline)
.subscriptionStorePickerItemBackground(.thinMaterial)
.storeButton(.visible, for: .redeemCode)
```

![storeKit][storeKit]


[storeKit]: WWDC23-10148-storeKit


```swift
import SwiftUI
import StoreKit

struct SubscriptionStore_Snippet {
    var body: some View {
        SubscriptionStoreView(groupID: passGroupID) {
            MyMarketingContent()
                .lightMarketingContentStyle()
                .containerBackground(for: .subscriptionStoreFullHeight) {
                    SkyBackground()
                }
        }
        .backgroundStyle(.clear)
        .subscriptionStoreButtonLabel(.multiline)
        .subscriptionStorePickerItemBackground(.thinMaterial)
        .storeButton(.visible, for: .redeemCode)
    }
}
```

See more in the session:  

[Meet StoreKit for SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10013)

# Simplified data flow

![Overview of the section titled 'Simplified data flow'][Simplified-data-flow]

[Simplified-data-flow]: WWDC23-10148-Simplified-data-flow


- `@Observable` macro
    - familiar patterns while keeping code precise and performant
    - simply add `@Observable` to a class
    - no need for `@State` or similar things in the view
    - only read values trigger view updates
    - get rid of `@`ObservableObject, `@StateObject` and others, only `@State` and `@Environment` are left


```swift
// Observable model
import Foundation

@Observable
class Dog: Identifiable {
    var id = UUID()
    var name = ""
    var age = 1
    var breed = DogBreed.mutt
    var owner: Person?
}
```
- `@Observable` macro
    - And this view is reading the isFavorite property, so when that changes, it will get reevaluated. Invalidation only happens for properties which are read, so you can pass your model through intermediate views without triggering any unnecessary updates.

```swift
struct DogCard: View {
    var dog: Dog
    
    var body: some View {
        DogImage(dog: dog)
        .overlay(alignment: bottom) {
            HStack {
                Text(dog.name)
                Spacer()
                Image(systemName: "heart")
                    .symbolVariant(
                        dog.isFavorite ? .fill : .none)
            } 
            .background(.thinMaterial)
        }
    }
}
```

![observable][observable]

[observable]: WWDC23-10148-observable

- `@State` can be passed into the environment
    - can then be read either via the type (e.g. `User.self`) or with a custom key

SwiftUI includes several tools for defining your state and its relationship to your views, several of which are designed for use with ObservableObject:
- @State
- @StateObject
- @ObservedObject
- @Environment
- @EnvironmentObject

When using Observable, this is becomes even simpler since it's designed to work directly with the State and Environment dynamic properties:
- @State
- @Environment

Observables are also a natural fit to represent mutable state, like on this form for a new dog sighting. The model is defined using the State dynamic property, and I'm passing bindings to its properties to the form elements responsible for editing that property.
```swift
@State private var model = DogDetails ()
    
    Form {
        Section {
            TextField( "Name"', text: $model.dogName)
            DogBreedPicker(selection: $model.dogBreed)
        }
        Section {
            StarRating(selection: $model.rating)
            TextField("Location", text: $model.location)
        }
        Section {
            PhotosPicker(
                "Add photo...", selection: $model.photo, 
                matching: .images)
                }
        }
    }
```

![state example][state]

[state]: WWDC23-10148-state

Lastly, Observable types integrate seamlessly into the environment.
```swift
@main
struct WhatsNew2023: App {
    @State private var currentUser: User?
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(currentUser)
            }
        }
}

@Observable final class User { ... }
```

Since views throughout our app want a way to fetch the current user, I've added it to the environment of my root view. The user profile view then reads the value using the Environment dynamic property. I'm using the type as the environment key here, but custom keys are also supported.
```swift
struct ProfileView: View {
    @Environment (User.self) private var currentUser: User?
    
    var body: some View {
        if let currentUser {
            UserDetails(user: currentUser)
        } else {
            LoginScreen ()
        }
    }
}
```

Be sure to watch:

[Discover Observation in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10149)

### SwiftData

#### `@Model` instead of `@Observable` (but gets its features)
To set up my Dog model type for SwiftData, I'll switch from using Observable to the Model macro. This is the only change I need to make. In addition to the persistence provided by SwiftData, models also receive all the benefits of using Observable.
```swift
// SwiftData model
import Foundation 
import SwiftData

@Model
class Dog {
    var name = ""
    var age = 1
    var breed = DogBreed.mutt
    var owner: Person?
}
```

#### the changes needed to use SwiftData
- add a modifier for `.modelContainer` with the definition of the model type (`.modelContainer(for: Dog.self)`) added to the `WindowGroup` of the app
- inside of the view, switch `@State` to `@Query` which allows e.g. sorting (`@Query(sort: \.dateSpotted)`) and efficient data loading even for large datasets
    - also stores document data

```swift
import SwiftUI 
import SwiftData

@main
struct WhatsNew2023: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        modelContainer (for: Dog.self)
    }
}
```

Then in my view code, I'll switch my array of dogs to use the new Query dynamic property instead of @State. Using Query will tell SwiftData to fetch the model values from the underlying database.
```swift
import SwiftUI
import SwiftData

struct RecentDogs: View {
    @Query private var dogs: [Dog] = ...
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(dogs) { dog in
                    DogCard (dog: dog)
                }
            }
        }
    }
}
```
Query is incredibly efficient for large data sets and allows for customization in how the data is returned, such as changing the sort order to use the date I spotted the dog:
```swift
struct RecentDogs: View {
    @Query(sort: \.dateSpotted) private var dogs: [Dog]
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(dogs) { dog in
                    DogCard (dog: dog)
                }
            }
        }
    }
}
```

#### DocumentContainer
- DocumentContainer gets updates
    - SwiftData
    - Sharing options
    - Undo/Redo support
    - Inspector (presenting differently on the respective layouts)

```swift
// DocumentGroup with SwiftData model
import SwiftUI 
import SwiftData

@main
struct WhatsNew2023: App {
    var body: some Scene {
        DocumentGroup(editing: DogTag.self, contentType: •dogTag) {
            ContentView()
        }
    }
}

extension UTType {
    static var dogTag: UTType {
        ...
    }
}
```

Please see:

[Meet SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10187)

and 

[Build an app with SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10154)

### DocumentGroup
DocumentGroup also gains a number of new platform affordances when running on iOS 17 or iPadOS 17, such as automatic sharing and document renaming support, as well as undo controls in the toolbar.

### Inspector
Inspector is a new modifier for displaying details about the current selection or context. It's presented as a distinct section in your interface. On macOS, Inspector presents as a trailing sidebar. as well as on iPadOS in a regular size class. In compact size classes, it will present itself as a sheet.

```swift
// Inspector
struct ContentView: View {
    @State private var inspectorPresented = true

    var body: some View {
        DogTagEditor()
            .inspector(isPresented: $inspectorPresented) {
            DogTagInspector()
        }
    }
}
```
![inspector][inspector]

[inspector]: WWDC23-10148-inspector

![inspector][inspector3]

[inspector3]: WWDC23-10148-inspector3

![inspector][inspector4]

[inspector4]: WWDC23-10148-inspector4


To uncover all the details on Inspector, watch:

[Inspectors in SwiftUI: Discover the details - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10161)

### Dialogs
New modifiers to give my image export dialog some useful information, like adjusting the confirmation button's label.

```swift
ContentView()
    .fileExporter(isPresented: $isExporterPresented, 
        item: selectedItem, 
        contentType: .png, 
        defaultFilename: selectedItem?.name)
{
    handleDataExport($0)
}
.fileExporterFilenameLabel("Export Image")
.fileDialogConfirmationLabel("Export Image")
```

![dialog][dialog]

[dialog]: WWDC23-10148-dialog

- Dialogs get new powers
    - e.g. severity, suppression toggle
    - HelpLinks


```swift
// Dialog customizations

DogTagEditor()
    .confirmationDialog(
        "Are you sure you want to delete the selected dog tag?", 
        isPresented: $showDeleteDialog)
{
Button("Delete dog tag", role: .destructive) {
    deleteSelectedTag()
    }
    HelpLink {
     ..
    }
} 
.dialogSeverity(.critical)
.dialogSuppressionToggle(isSuppressed: $suppressed)
```

![dialog][dialog2]

[dialog2]: WWDC23-10148-dialog2

### Lists and tables
- Lists and Tables
    - customization of data toggling
    - programmatically expand sections
    - new stylings for tables

Lists and tables are a key part of most apps, and SwiftUI has brought some new features and APIs for fine-tuning them in iOS 17 and macOS Sonoma. Tables support customization of their column ordering and visibility. When coupled with the SceneStorage dynamic property, these preferences can be persisted across runs of your app. You provide the table with a value representing the customization state and give each column a unique stable identifier.
```swift
// Table customization support
struct DogSightingsTable: View {
    @SceneStorage ("TableConfiguration")
    private var columnCustomization: TableColumnCustomization<DogSighting>
    
    var body: some View {
        Table(dogSightings, selection: $selectedSighting,
            columnCustomization: $columnCustomization) {
            TableColumn ("Dog Name", value: \.name)
                .customizationID("name")
            TableColumn("Dog Breed", value: \.breed.name)
                .customizationID("breed")
                
            ...
            
        }
    }
}
```
Tables now also have all the power of OutlineGroup built in. This is great for large data sets that lend themselves to a hierarchical structure, like this one which groups some of my favorite dogs with their proud parents. Simply use the new DisclosureTableRow to represent rows that contain other rows, and build the rest of your table as you would normally.

```swift
Table (of: DogGenealogy.self) {
    TableColumn ("Dog Name", value: \.name)
    TableColumn ("Dog Breed", value: \.breed.name)
    TableColumn( "Age (Dog Years)") {
        Text ($0.age, format: .number)
    }
    TableColumn ("Favorite Toy", value: \.favoriteToy)
} rows: {
    ForEach(dogs) { dog in
        DisclosureTableRow(dog) {
            ForEach(dog.children) { child in
                TableRow(child)
            }
        }
    }
}
```

![tables][tables]

[tables]: WWDC23-10148-tables


### Sections
Sections within a list or table have gained support for programmatic expansion. I've used it here in my app's sidebar to show the location section as collapsed initially, but while still allowing for expansion. The new initializer takes a binding to a value which reflects the current expansion state of the section.
```swift
// Programmatic Section expansion
Section("Dog Breeds", isExpanded: $isBreedSectionExpanded) {
    DogBreeds()
}
Section ("Locations", isExpanded: SisLocationSectionExpanded) {
    SightingLocations()
}
```

![sections][sections]

[sections]: WWDC23-10148-sections

For smaller data sets, tables have also gained a few new styling affordances, such as how row backgrounds and column headers are displayed.
```swift
Table(dogSightings, selection: $selection) {
    TableColumn("Breed" ) {
        Text(SO.name)
    }
    TableColumn("Sightings") {
        Text($0.sightings, format: .number)
    }
    TableColumn("Rating") {
        StarRating($0.rating)
    }
}
.alternatingRowBackgrounds(.disabled)
.tableColumnHeaders(.hidden)
```

Custom controls like my star rating will also benefit from the new background prominence environment property. Using a less prominent foreground style when the background is prominent lets my custom control feel right at home in a list.
```swift
StarRating(sighting.rating)
    .foregroundStyle(.starRatingForeground)

struct StarRatingForegroundStyle: ShapeStyle {
    func resolve(in environment: EnvironmentValues)
    -> some ShapeStyle
{
        if environment.backgroundProminence == .increased {
            return AnyShapeStyle(.secondary)
        } else {
            return AnyShapeStyle(.yellow)
        }
    }
}
```

![sections][section2]

[section2]: WWDC23-10148-section2

We've also made big improvements to the performance, particularly when dealing with large data sets. To learn more about this and the ways you can optimize your own SwiftUI views, check out:

[Demystify SwiftUI performance - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10160)

# Extraordinary new animations APIs

![Overview of the section titled 'Extraordinary animations'][Extraordinary-animations]

[Extraordinary-animations]: WWDC23-10148-Extraordinary-animations

- Keyframe Animator API
    - animate multiple values in parallel
    - give the animator a value of animatable values
    - `KeyframeAnimator` defines a view, then a list of `KeyframeTrack`s with different keyframe animations.

Changes to the state trigger my animation. In the first closure, I build a view, modified by my animatable properties, like the vertical offset of my logo. In the second closure, I define how these properties change over time. For example, the first track defines the animation of my verticalTranslation property. I pull my logo down 30 points over the first quarter second using a spring animation. Then I make my Beagle leap and land using a cubic curve. Finally, I bring this dog home with a natural spring animation. I define additional tracks for my other animated properties.  
All these tracks run in parallel to create this cool animation. To learn how to leverage keyframe animators in your apps, check out:

[Wind your way through advanced animations in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10157)

```swift
KeyframeAnimator(
    initialValue: LogoAnimationValues(), trigger: runPlan
) { values in
    LogoField(color: color, isFocused: isFocused)
        .scaleEffect(values.scale)
        .rotationEffect(values.rotation, anchor: .bottom)
        .offset(y: values.verticalTranslation)
} keyframes: { _ in
	KeyframeTrack(\.verticalTranslation) {
		SpringKeyframe(30, duration: 0.25, spring: .smooth)
		CubicKeyframe(-120, duration: 0.3)
		CubicKeyframe(-120, duration: 0.3)
		SpringKeyframe(0, spring: .bouncy)
	}

	KeyframeTrack(\.scale) { ... }
	KeyframeTrack(\.rotation) { ... }
```

### Phase Animator
- difference to keyframe: sequentially go through animation steps, while keyframe goes through multiple ones in parallel
- start one animation when the previous one has finished

Ex. working on an Apple Watch app to record dog sightings. It's pretty simple so far, just our happy icon and a button to register a sighting. I'd like to animate this icon when I tap the button. This is a good place for a phase animator. A phase animator is simpler than a keyframe animator. Instead of parallel tracks, it steps through a single sequence of phases. This lets me start one animation when the previous animation finishes. I give the animator a sequence of phases and tell it to run my animation whenever my sightingCount changes. Then in this first closure, I set the rotation and scale of my happy dog based on the current phase. The second closure tells SwiftUI how to animate into each phase.
They're now the default animation for apps built on or after iOS 17 and aligned releases.  
Haptic feedback is easy with the new sensory feedback API. To play haptic feedback, I just attach the sensoryFeedback modifier, specify what sort of feedback I want and when it should happen.

```swift
HappyDog()
    .phaseAnimator(
        SightingPhases.allCases, trigger: sightingCount
    ) { content, phase in
        content
            .rotationEffect(phase.rotation)
            .scaleEffect(phase.scale)
    } animation: { phase in
        switch phase {
        case .shrink: .snappy(duration: 0.1)
        case .spin: .bouncy
        case .grow: .spring(
            duration: 0.2, bounce: 0.1, blendDuration: 0.1)
        case .reset: .linear(duration: 0.0)
        }
    }
    .sensoryFeedback(.increase, trigger: sightingCount)
```

![Phase Animator][phaseAnimator]

[phaseAnimator]: WWDC23-10148-phaseAnimator

- define multiple phases and hand them to the `.phaseAnimator` modifier, together with a trigger that triggers the animation whenever a value changes
- define the content and how the elements of the phases change it (e.g. rotation, scale)
- provide the types of animations for the different phases, e.g. `.snappy`, `.bouncy`, or new `spring` options
- Haptic Feedback
    - `.sensoryFeedback(.increase, trigger: sightingCount)`

Check out the Human Interface Guidelines to learn what sorts of feedback will be best in your apps:

[Playing haptics - Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/playing-haptics)

To learn about the fundamentals of animation in SwiftUI, check out:

[Explore SwiftUI animation - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10156)  

and

[Animate with springs - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10158)  
 
### The visual effects modifier 

- works with coordinate spaces
- and geometry reader somehow
- sure fun to play around with

The visual effects modifier lets me update these dog photos based on their position. And I don't need a GeometryReader to do it. I've got a little simulation that moves a focal point around the screen. This red dot shows what I mean by focal point. I associate a coordinate space with this grid that shows all the dogs.

```swift
// Coordinate spaces and visual effects
ScrollView {
	LazyVGrid(columns: columns) {
		ForEach(dogs) { dog in
			DogCircle(dog: dog, focalPoint: simulation.point)
		}
	}	
}
.coordinateSpace(.dogGrid)
```

Then inside my DogCircle view, I add a visual effect. The closure gets my content to modify and a geometry proxy. I'm passing the geometry proxy to a helper method to compute the scale.
```swift
// Coordinate spaces and visual effects
struct DogCircle: View {
	var dog: Dog 
	var focalPoint: CGPoint
	
	var body: some View {
		DogImage (dog: dog)
			.visualEffect { content, geometry in 
				content
					.scale(contentScale(in: geometry))
					.grayscale(contentGrayscale(in: geometry))
					.saturation(contentSaturation (in: geometry))
			}
		}
	}
}
```

![Visual Effects Modifier][visualEffectsModifier]

[visualEffectsModifier]: WWDC23-10148-visualEffectsModifier


I can use the geometry proxy to get the size of my grid view and the frame of a single dog circle relative to my grid view. That lets me compute how far any dog is from the focal point of the simulation, so I can scale up the focused doggos. With visual effects, I can do all of this without using a GeometryReader. And it automatically adapts to different sizes.
```swift
// Coordinate spaces and visual effects
func contentScale(in geometry: GeometryProxy) -> Double {
guard let gridSize = geometry.bounds (of: .dogGrid)?.size else { return 0 } 
let frame = geometry.frame(in: .dogGrid)
...
}
```


### Style Text
- Text can now be styles with a `foregroundStyle` inside another Text
- `Text("\(Text(dog.name).foregroundStyle(stripes)) is a good dog")`
- It works with a Shader
- Can bring Metal shaders into SwiftUI with ShaderLibrary

I'm passing my stripeSpacing and angle, along with a color from my asset catalog, to a custom Metal shader.

```swift
var stripes: Shader {
	ShaderLibrary.angledFill(
		.float(stripeSpacing),
		.float(stripleAngle),
		.color(Color(.stripes))
	)
}

var stripes: Shader {
	ShaderLibrary.angledFill(
		.float(stripeSpacing),
.		.float(stripeAngle), 
		.color(Color(.stripes))
	)
}
```

![Text Styles][textStyles]

[textStyles]: WWDC23-10148-textStyles

Using SwiftUI's new ShaderLibrary, I can turn Metal shader functions directly into SwiftUI shape styles, like this one that renders the stripes in Furdinand's name.
```swift
// Metal shaders
[[ stitchable ]] half4
angledFill(float2 position, float width, float angle, half4 color)
{
	float pMagnitude = sqrt(position.x * position.x + position.y * position.y);
	float pAngle = angle +
		(position.x == 0.0f ? (M_PI_F / 2.0f) : atan(position.y / position.x));
	float rotatedX = pMagnitude * cos(pAngle);
	float rotatedY = pMagnitude * sin(pAngle);
	return (color + color * fmod(abs(rotatedX + rotatedY), width) / width) / 2;
}
```
If you'd like to take Metal shaders out for a spin, just add a new Metal file to your project and call your shader function using ShaderLibrary in SwiftUI.

### Symbol effect modifier
Symbols get a `.symbolEffect` modifier with multiple options:
- `.pulse`
- `.variableColor`
- `.scale`
- `.appear/disappear`
- `.replace`
- event notifications with `bounce`
- new `.textScale(.secondary)` modifier automatically scaling

```swift
Image(systemName: "..").symbolEffect(...)
```

![Symbol effect modifier][symbolEffectModifier]

[symbolEffectModifier]: WWDC23-10148-symbolEffectModifier


Check also:

[Animate symbols in your app - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10258)  

### New textScale modifier
I want to point out one last feature. Notice the units on the text here. In the past I might have used small caps for this effect, but now I can get this appearance by applying the new textScale modifier to my units.  
And, if Jeff and I bring our app to the Chinese market, the units will be sized correctly, even though the concept of small caps isn't part of the typography in Chinese.

```swift
// Text style
Text("\(space) \(Text("PX").textScale(.secondary))")
Text ("\(angle) \(Text("RAD").textScale(.secondary))")
```
![New textScale modifier][textScaleModifier]

[textScaleModifier]: WWDC23-10148-textScaleModifier

### The typesettingLanguage modifier
We have another tool to help apps work great in multiple locales. Some languages, like Thai, use taller letter forms. When text from one of these languages is embedded in text localized in a language with shorter letter forms, like English, the taller text can be crowded or clipped. When we know that this might be an issue -- for example, if our dog names were globally crowd-sourced -- we can apply the typesettingLanguage modifier. This lets SwiftUI know that the text might need more space.
```swift
// Typesetting language
struct Dog {
	var name: String 
	var language: Locale.Language
...
}

var dog = Dog(
	name: "lala", 
	language: .init(languageCode: .thai))

Text(
	"""
	Who's a good dog, \
	\(Text(dog.name).typesettingLanguage(dog.language))?
	""")
```

![New typesettingLanguage modifier][typesettingLanguage]

[typesettingLanguage]: WWDC23-10148-typesettingLanguage


# Enhanced interactions

![Overview of the section titled 'Enhanced interactions'][4]

[4]: WWDC23-10148-Enhanced-interactions

### Scroll Transition Effect


- the `.scrollTransition` modifier can be applied to elements inside of the ScrollView
- gets a `content` and a `phase`, let’s  apply effects to the `content` with the help of the `phase` properties, e.g. the `.isIdentity`
- `.containerRelativeFrame` allows to split the view into parts of the frame (`count` ) and define how much each element should span (`span`)
- can use a `.safeAreaInset` modifier with an edge to position this
- `.scrollTargetLayout` can be aded to the `LazyHStack` and then the `ScrollView` can get a `scrollTargetBehavior(.viewAligned)` modifier
- paging behavior also possible, or something custom using the `ScrollTargetBehavior` protocol
- `.scrollPosition` shows the top most item

I'd like to add some visual effects to my dog cards as they transition in and out of the visible area of my scroll view. The scroll transition modifier is very similar to the visual effect modifier Curt used earlier for the welcome screen. It lets you apply effects to items in your scroll view.

```swift
// Scroll transition effects
ScrollView {
	LazyVStack {
		ForEach(dogs) { dog in
			DogCard (dog: dog)
				.scrollTransition { content, phase in
			content
				.scaleEffect(phase.isIdentity ? 1 : 0.6) 
				.opacity (phase.isIdentity ? 1: 0)
			}
		}
	}
}
.safeAreaPadding(.horizontal, 16.0)
```

![Scroll Transition Effect][scrollTransitionEffect]

[scrollTransitionEffect]: WWDC23-10148-scrollTransitionEffect

I'd also like to add a side-scrolling list of my favorite dog parks to this screen. Above my vertical stack of dogs, I'll drop in a horizontal stack for the park cards. I'm using the new containerRelativeFrame modifier to size these park cards relative to the visible size of the horizontal scroll view. The count specifies how many chunks to divide the screen into. The span says how many of those chunks each view should take.This is pretty great, but I'd like my park cards to snap into place. The new scrollTargetLayout modifier makes that easy. I'll add it to the LazyHStack and modify the scroll view to align to views in the targeted layout.

```swift
// ScrollView layout enhancements
ScrollView {
	...
}
.safeAreaInset(edge: .top) {
	ScrollView(.horizontal) {
		LazyHStack {
			ForEach(parks) { park in
				ParkCard(park: park)
					.containerRelativeFrame(
					.horizontal, count: 5, span: 2, spacing: 8)
			}
		}
		.scrollTargetLayout ()
	} 
	.scrollTargetBehavior(.viewAligned)
}	
```

![Scroll Transition Effect][scrollTransitionEffect2]

[scrollTransitionEffect2]: WWDC23-10148-scrollTransitionEffect2

Scroll views can also be defined to use a paging behavior. And you can define your own behavior using the scrollTargetBehavior protocol. The new scrollPosition modifier takes a binding to the topmost item's ID, and it's updated as I scroll.
```swift
// Scroll position
ScrollView {
	...
}
safeAreaInset(edge: .top) {
	...
}
.scrollPosition(id: $scrolledID)

```

![Scroll Transition Effect][scrollTransitionEffect3]

[scrollTransitionEffect3]: WWDC23-10148-scrollTransitionEffect3

To learn more about all these and the other great improvements to Scroll View, be sure to watch:

[Beyond scroll views - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10159)  

### HDR images
- Images support HDR with `.allowDynamicRange(.high)` - use sparingly

Image now supports rendering content with high dynamic range. By applying the allowedDynamicRange modifier, the beautiful images in our app's gallery screen can be shown with their full fidelity. It's best to use this sparingly, though, and usually when the image stands alone. 
```swift
// HDR images
struct GalleryDetail: View {
	var dog: Dog
	
	var body: some View {
		VStack {
			CloseButton ()
			Spacer ()
			
			AsyncImage (url: dog.photoURL)
		
			Spacer()
			ImageActions()
		}
	}	
}		
```

![HDR images][HDRImages]

[HDRImages]: WWDC23-10148-HDRImages

### new accessibility APIs
I'm also going to add the new accessibilityZoomAction modifier to my view. This allows assistive technologies like VoiceOver to access the same functionality without using the gesture. I'll just update the zoom level depending on the action's direction, and I can see what mischief she's been up to now. VoiceOver: Zooming image view. Image.

```swift
// Accessibility zoom
DogImage(dog: dog)
	.scaleEffect (zoomLevel)
	.gesture (magnification)
	.accessibilityZoomAction { action in
		switch action.direction {
		case .ZoomIn:
			zoomLevel += 0.5 
		case .zoomOut:
			zoomLevel -= 0.5
	}
}
```

For more, be sure to check out:

[Build accessible apps with SwiftUI and UIKit - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10036)  

### Static member syntax for custom colors
- use static imports of colors defined in the asset catalog with `Color(.tennisBallYellow)`

Color now supports using static member syntax to look up custom colors defined in your app's asset catalog. This gives compile-time safety when using them, so you'll never lose time to a typo. For the document app I showed earlier, I've added a menu containing several useful actions to the toolbar. The top section of the menu is a ControlGroup with the new compactMenu style, which shows its items as icons in a horizontal stack. The tag color selector is defined as a picker with the new palette style. Using this style in concert with symbol images gives a great visual representation in menus, especially one like this where I can use the label's tint to differentiate them.

```swift
// Static member syntax for custom colors
Picker("Tag Color", selection: $selection) {
	Label ("Tennis Ball Yellow")
		.tint (Color(.tennisBallYellow))
		.tag(.yellow)
	Label("Rawhide Brown")
		.tint(Color(.rawhideBrown))
		.tag (.brown)
		
		...
}
```

![Static Colors][staticColors]

[staticColors]: WWDC23-10148-staticColors


Lastly, the paletteSelectionEffect modifier lets me use a symbol variant to represent the selected item in the picker. With my menu in place, Buddy's dog tag can now be his favorite color, tennis-ball yellow.

```swift
Picker("Tag Color", selection: $selectedTagColor) {
	ForEach(tagColors) { tagColor in
		Label(tagColor.title, systemImage: "tag")
			.tint(tagColor.color)
			.tag(tagColor)
	}
} 
.pickerStyle(.palette)
.paletteSelectionEffect(.symbolVariant(.fill))
```

![Static Colors][staticColors2]

[staticColors2]: WWDC23-10148-staticColors2



### The new compactMenu style
- Control group with new compactMenu style (`.controlGroupStyle(.compactMenu)`)
- Pickerstyle with a `.pickerStyle(.palette)` and the `.paletteSelectionEffect(.symbolVariant(.fill))`

The top section of the menu is a ControlGroup with the new compactMenu style, which shows its items as icons in a horizontal stack. The tag color selector is defined as a picker with the new palette style. Using this style in concert with symbol images gives a great visual representation in menus, especially one like this where I can use the label's tint to differentiate them. Lastly, the paletteSelectionEffect modifier lets me use a symbol variant to represent the selected item in the picker.

```swift
ControlGroup {
	Button {
		cutSelection()
	} label: {
		Label("Cut", systemImage: "scissors")
	}
	Button {
		copySelection ()
	} label: {
		Label ("Copy", systemImage: "doc.on.doc")
	}
}
.controlGroupStyle (.compactMenu)
```

![The new compactMenu style][controlGroupStyle]

[controlGroupStyle]: WWDC23-10148-controlGroupStyle

The tag color selector is defined as a picker with the new palette style. Using this style in concert with symbol images gives a great visual representation in menus, especially one like this where I can use the label's tint to differentiate them. Lastly, the paletteSelectionEffect modifier lets me use a symbol variant to represent the selected item in the picker.

```swift
Picker ("Tag Color", selection: $selectedTagColor) {
	ForEach(tagColors) { tagColor in
		Label(tagColor.title, systemImage: "tag")
			.tint(tagColor.color)
			.tag (tagColor)
	}	
} 
.pickerStyle(.palette)
.paletteSelectionEffect(.symbolVariant(.fill))
```

![The new compactMenu style][controlGroupStyle2]

[controlGroupStyle2]: WWDC23-10148-controlGroupStyle2


### Bordered buttons
- New Button styles (coming after `.buttonStyle(.bordered)`)
    - `.buttonBorderShape(.roundedRectangle)`
    - `buttonBorderShape(.circle)`
- Buttons can support drag actions / force-clicking (macOS) with the `.springLoadingBehavior(.enabled)` modifier

![Bordered buttons][borderedButtons]

[borderedButtons]: WWDC23-10148-borderedButtons

Bordered buttons can now be defined with new built-in shapes, such as circle and rounded rectangle. These new border shape styles work on iOS, watchOS, and macOS.
```swift
// Bordered button shapes
Button {
	toggleGuides()
} label: {
	Label("Toggle Guides", systemImage: "ruler")
}
.buttonStyle(.bordered)
.buttonBorderShape(. roundedRectangle)

...

Button {
	showPopover.toggle()
} label: {
	Label("Add", systemImage: "plus")
}
.buttonStyle(.bordered)
.buttonBorderShape (.circle)
```


Buttons on macOS and iOS can now react to drag actions, like this button in my editor that opens a popover.
The new springLoadingBehavior modifier indicates that a button should trigger its action when a drag pauses over it, or when force-clicking it on macOS.
```swift
// Button spring-loading behavior

Button {
	showPopover = true
} label: {
	Label("Add", systemImage: "plus")
.buttonStyle(.bordered)
.buttonBorderShape(.circle)
.springLoadingBehavior(.enabled)
```

![Bordered buttons][borderedButtons2]

[borderedButtons2]: WWDC23-10148-borderedButtons2

### New highlight hover effect (tvOS)
- New `.hoverEffect(.highlight)` (only available for tvOS?)

```swift
// Highlight hover effect

struct DogGalleryCard: View {
	@FocusState private var isFocused: Bool 
	var dog: Dog
		
	var body: some View {
		Button {
			openDogDetail(dog)
		} label: {
			DogGalleryImage(dog: dog)
				.hoverEffect(.highlight)
			Text(dog.name)
				.opacity(isFocused ? 1 : 0)
		} 
		.buttonStyle(.borderless)
		.focused (SisFocused)
	}
}
```


### Handling Key Presses 
- `.onKeyPress` to allow for reaction to any keyboard input
Focusable views on platforms with hardware keyboard support can use the onKeyPress modifier to directly react to any keyboard input. The modifier takes a set of keys to match against and an action to perform for the event.

```swift
// Handling key presses
DogTagEditor()
	.focusable (true, interactions: •edit)
	.focusEffectDisabled ()
	.onKeyPress (characters: letters, phases: .down) { press in
		...
}
```

To get your fill of focus-related recipes, be sure to watch

[The SwiftUI cookbook for focus - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10162)
  
# Wrap up
- SwiftUl in more places  
- Simplified data flow  
- Extraordinary animations  
- Enhanced interactions  

## Resources

[Have a question? Ask with tag wwdc2023-10148](https://developer.apple.com/forums/create/question?tag1=755030&tag2=239)  
[Search the forums for tag wwdc2023-10148](https://developer.apple.com/forums/tags/wwdc2023-10148)  
[Playing haptics](https://developer.apple.com/design/Human-Interface-Guidelines/playing-haptics)  

# Related Videos
[Animate symbols in your app - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10258)  
[Animate with springs - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10158)  
[Beyond scroll views - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10159)  
[Bring widgets to new places - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10027)  
[Bring widgets to life - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10028/)  
[Build accessible apps with SwiftUI and UIKit - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10036)  
[Build an app with SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10154)  
[Build programmatic UI with Xcode Previews - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10252)  
[Design and build apps for watchOS 10 - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10138)  
[Demystify SwiftUI performance - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10160)  
[Discover Observation in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10149)  
[Explore pie charts and interactivity in Swift Charts - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10037)  
[Explore SwiftUI animation - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10156)  
[Inspectors in SwiftUI: Discover the details - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10161)  
[Meet MapKit for SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10043)  
[Meet StoreKit for SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10013)  
[Meet SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10187)  
[Meet SwiftUI for spatial computing - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10109)  
[The SwiftUI cookbook for focus - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10162)  
[What’s new in Swift - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10164)  
[Wind your way through advanced animations in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10157)  
[Update your app for watchOS 10](https://developer.apple.com/videos/play/wwdc2023/10031)  
