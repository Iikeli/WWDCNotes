# Build an app with SwiftData

Discover how SwiftData can help you persist data in your app. Code along with us as we bring SwiftData to a multi-platform SwiftUI app. Learn how to convert existing model classes into SwiftData models, set up the environment, reflect model layer changes in UI, and build document-based applications backed by SwiftData storage.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10154", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
      @GitHubUser(multitudes)
   }
}



## Chapters
[0:00 - Meet the app](https://developer.apple.com/wwdc23/10154)  
[3:09 - SwiftData models](https://developer.apple.com/wwdc23/10154?time=189)  
[5:30 - Querying models to display in UI](https://developer.apple.com/wwdc23/10154?time=330)  
[9:43 - Creating and updating](https://developer.apple.com/wwdc23/10154?time=583)  
[12:30 - Documents based apps](https://developer.apple.com/wwdc23/10154?time=750)

# Intro

![SwiftData Logo][SwiftData]  

[SwiftData]: WWDC23-10154-SwiftData

To cover your basics, watch the "Meet SwiftData" session first if you haven’t already.

[Meet SwiftData - WWDC23](https://developer.apple.com/wwdc23/10187)  

This is a code-along. The sample project includes both the starting point and how the project should look after the session.

![Code Along][codeAlong]  

[codeAlong]: WWDC23-10154-codeAlong

Download the sample project here: [Code Along - Building a document based app using SwiftData](https://developer.apple.com/documentation/SwiftUI/Building-a-document-based-app-using-SwiftData)

# Meet the app
Let’s build a flashcards app. This app should work everywhere: on Mac, iPhone, Watch, and TV.

![Code Along][codeAlong2]  

[codeAlong2]: WWDC23-10154-codeAlong2

There is a new Xcode feature, embedded interactive live Previews for Mac.

In the Previews section, there's a grid with some flash cards. The app is populated with sample cards stored in memory, and if we run the app and add new ones, they will disappear when we close the app.

This is where SwiftData comes in.

## SwiftData models
In the starter project, there is a Card model that represents a single flash card, some views, and supporting files. Every card stores the text for the front and back sides and the creation date.


![Code Along][codeAlong3]  

[codeAlong3]: WWDC23-10154-codeAlong3

It is a pretty typical model.

First, import SwiftData into this file. Adding the @Model macro to the definition.

With @Model, the Card gets conformance to the Observable protocol, and we will use it instead of ObservableObject. Remove the conformance to the Observable object as well as @Published property wrappers.

![Code Along][codeAlong4]  

[codeAlong4]: WWDC23-10154-codeAlong4

To adopt Observable in the CardEditorView file, replace the "ObservedObject" property wrapper with "Bindable." It allows the text fields to bind directly to the card's front... and back text.

![Code Along][codeAlong5]  

[codeAlong5]: WWDC23-10154-codeAlong5


## @Observable
- Set up data flow with less code  
- Automatic dependencies  
- Seamlessly bind models' mutable state to UI  

When a View uses a property of an Observable type in its body, it will be updated automatically when the given property changes.

See more in the session:
[Discover Observation in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10149)

## Querying models to display in UI

Let’s switch to ContentView and display the cards stored till now.  
And replace @State property wrapper with @Query.  Use @Query whenever you want to display models, managed by SwiftData, in the UI.

![Code Along][codeAlong6]  

[codeAlong6]: WWDC23-10154-codeAlong6

## @Query (Property wrapper)
- Provides the view with data
- Triggers view update on every change of the models
- A view can have multiple `@Query` properties
- Uses `ModelContext` as the source of data

@Query is a new property wrapper that queries the models from SwiftData. It also triggers the view updated on every change of the models, similarly to how @State would do that.

```swift
@Query(sort: \.created) private var cards: [Card]
```

Every view can have many queried properties. Query offers lightweight syntax to configure sorting, ordering, filtering, and even animating changes.

`@Query` needs a model context for it to work. We get the model context from the model container.

## modelContainer (View modifier)

```swift
.modelContainer(for: Card.self)
```

- Set up the model container
- Create the storage stack
- Each view has a single model container

An app needs to set up at least one model container. Model containers are inherited in by the views (like `.environment()`). It creates the whole storage stack, including the context that @Query will use.

A View has a single model container, but an application can create and use as many containers as it needs for different view hierarchies.

```swift
@main
struct FlashCardApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer (for: Card.self)
    }
}
```

![Code Along][codeAlong7]  

[codeAlong7]: WWDC23-10154-codeAlong7

In this case, we can set it up for the whole window group scene. The window and its views will inherit the container, as well as any other windows created from the same group. All of these views will write and read from a single container.
Some apps need a few storage stacks, and they can set up several model containers for different windows.

```swift
@main
struct FlashCardApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Card.self)
        
        Window("Card Designer") {
            CardDesignerView()
        }
        .modelContainer(for: Design.self)
    }
}
```

![Code Along][codeAlong8]  

[codeAlong8]: WWDC23-10154-codeAlong8

Different views in the same window can have separate containers, and saving in one container won’t affect another.

```swift
struct CardDesignInspector: View {
    var body: some View {
        VSplitView {
            Form { ... }
            ObjectLibrary()
                .modelContainer(for: Template.self)
        }
    }
}
```

![Code Along][codeAlong9]  

[codeAlong9]: WWDC23-10154-codeAlong9

Now, let's set up the modelContainer to provide the Query with a source of data. I open the app definition...  And set a model container for app's windows. Note that the subviews can create, read, update, and delete only the model types listed in the view modifier.

And let's provide the previews with sample data.

![Code Along][codeAlong10]  

[codeAlong10]: WWDC23-10154-codeAlong10

In the app, we have defined an in-memory container with sample cards. Open the "PreviewSampleData" file and include it in the target. This file contains the definition of a container with the sample data.

![Code Along][codeAlong11]  

[codeAlong11]: WWDC23-10154-codeAlong11

Now that @Query has a source of data, the preview displays the cards!

![Code Along][codeAlong12]  

[codeAlong12]: WWDC23-10154-codeAlong12

# Creating and updating

## modelContext (Environment variable)

- Provides access to the model context
- A view has a single model context

Next, to make sure that SwiftData tracks and saves the new cards, as well as the changes made to the existing ones, we need the model context of the view.

To access the model context, SwiftUI offers a new environment variable.

```swift
@Environment(\.modelContext) private var modelContext
```

This environment variable was populated automatically when the model container was set earlier.

We will need to access the modelContext to save and update the cards.

![Code Along][codeAlong13]  

[codeAlong13]: WWDC23-10154-codeAlong13

SwiftData autosaves the model context. The autosaves are triggered by UI-related events and user input. Call "save()" explicitly, when we want to make sure that all the changes are persisted immediately, but this only in some special cases.

# Document-based apps

## DocumentGroup

Document-based apps is a concept used on macOS, iOS, and iPadOS. It describes the certain types of applications that allow users to create, open, view, or edit different types of documents. Every document is a file, and users can store, copy, and share them.

![Code Along][codeAlong14]  

[codeAlong14]: WWDC23-10154-codeAlong14

Document-based apps exist on iOS and macOS, and on these platforms, we'll switch to using the DocumentGroup initializer. We will be passing in the model type Card.self, content type, and a view builder.

About the content type:

![Code Along][codeAlong15]  

[codeAlong15]: WWDC23-10154-codeAlong15

SwiftData Document-based apps need to declare custom content types. Each SwiftData document is built from a unique set of models and so has a unique representation on disk. In the context of documents, a content type is like a binary file format, ex JPEG. Another type of documents, a package, is a directory with a fixed structure on disk, like an Xcode project. For example, all the JPEG images have the same binary structure. Otherwise, photo editors wouldn’t know how to read them. Similarly, all the Xcode projects contain certain directories and files.

![Code Along][codeAlong16]  

[codeAlong16]: WWDC23-10154-codeAlong16

When the user opens the deck, we need the operating system to associate the deck format and file extension with our app. That’s why we need to declare the content type. SwiftData documents are packages: if you mark some properties of a SwiftData model with the “externalStorage” attribute, all the externally stored items will be a part of the document package.

In the UTType+FlashCards file, I have a definition of the new content type, so we can conveniently use it in code. We'll put the same definition in the Info.plist.  
We are about to declare a new content type in the operating system. We need to specify the file extension to help to distinguish the card decks created by our app from any other documents. For this sample app, we’ll use “sampledeck” as an extension.  
And add a short description, like Flash Cards Deck.  
The identifier should be exactly the same as the one in the code.  
Because SwiftData documents are packages, we have to make sure our type conforms to com.apple.package.

![Code Along][codeAlong17]  

[codeAlong17]: WWDC23-10154-codeAlong17

And now, returning to the app definition and passing the content type to the DocumentGroup. The view builder looks identical.  
We don’t set up the model container. The document infrastructure will set up one for each document.

![Code Along][codeAlong18]  

[codeAlong18]: WWDC23-10154-codeAlong18

A workflow: 
The app launches with the open panel, which is standard behavior for Document-based applications. We'll create a new document and add a card there. The document now has a toolbar subtitle indicating that it has unsaved changes. We press Command+S, and the save dialog appears. The deck will be saved with the same file extension that we put in the Info.plist. Can also press Command+N to create a new deck, or Command+O to open one. These shortcuts, as well as many other features, Document-based applications get automatically. 

Recap:
- You have to set up a custom `contentType`/file format for your documents.
- The content type has to be set up in Info.plist with the same identifier as you use in code.
- The content type set up in Info.plist needs to conform to `com.apple.package`.
- We should NOT set up an `.modelContainer` for the `DocumentGroup`, as it does it automatically.

# Resources
[Search the forums for tag wwdc2023-10154](https://developer.apple.com/forums/tags/wwdc2023-10154)  
[Have a question? Ask with tag wwdc2023-10154](https://developer.apple.com/forums/create/question?&tag1=719030&tag2=677030)  
[</> Building a document-based app using SwiftData](https://developer.apple.com/documentation/SwiftUI/Building-a-document-based-app-using-SwiftData)  

# Related Videos
[Meet SwiftData - WWDC23](https://developer.apple.com/wwdc23/10187)  
[Discover Observation in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10149)  
[Dive deeper into SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10196)  
[Migrate to SwiftData - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10189)  
[Model your schema with SwiftData -                WWDC23](https://developer.apple.com/videos/play/wwdc2023/10195)  
[What’s new in Swift - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10164)  
[What’s new in SwiftUI - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10148)  
