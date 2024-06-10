# Design and build apps for watchOS 10

Dive into the details of watchOS design principles and learn how to apply them in your app using SwiftUI. We’ll show you how to build an app for the redesigned user interface to surface timely information, communicate focused content at a glance, and make navigation consistent and predictable.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10138", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(arnoappenzeller)
   }
}



## Key design principles
* Main Question: what is the most relevant information of my app?
* Example new weather app
* Watch experience should be optimised on brief interaction
* Digital crown interactions should be always backed up by touch interaction
* Which information would be the core for a widget of my app
    * Build app around this foundation

## Navigation
* NavigationSplitView
    * Concept borrowed by two column layout
    * Perfect if app has source list
    * App should open directly in detail
    * Transition between source list and detail view
    * API similar to other plattforms
* TabView
    * Example: Activity app
    * Animations based on election view
        * matchedGeometryEffect for merging and animating content between tabs
* NavigationStack
    * Lead people into and back from hierarchical structure


## Layout
* Flexibel Grid
* 3 foundational layouts
    * List
        * Scroll though content
    * Dail
        * Dense information delivered at one glance
    * Infograhic
        * Ideal for data visualisation
* Topbar Leading and Topbar Trailing Placement
    * Time moves to center
* More control on screen by using less interaction time

## Color and materials
* Should give sense of view hierarchy
* Four background materials
    * Ultra Thin, Thin, regular, thick
* Use color to differentiate similar views
* Convey state change with color
* Vibrant versions of all system colors
* Example: Noise App
* NavigationBar with blur
