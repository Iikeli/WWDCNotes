# Introducing the Indoor Maps Program

The Indoor Maps Program enables organizations with large public or private spaces to deliver user experiences that leverage precise location information and present stunning indoor maps. Learn the entire enablement workflow including, creation of a standards-based map definition, map validation, testing and calibration, and details on how to use MapKit and MapKit JS to integrate it all into your app or website.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/245", purpose: link, label: "Watch Video (26 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- Enables public places to use indoor maps
- Uses a format called “Indoor Mapping Data Format” (IMDF) which is represented as a set of GeoJSON files.
- IMDF conforms to RFC 7959.
- The building owner has to join the “[Indoor Maps Program](http://register.apple.com/indoor)” and produce the IMDF.
- The IMDF map has to be created with a professional GIS or BIM tool.
- Anyone can create an IMDF map and display it an app or website using MapKit or MapKit JS.
- The Indoor Maps Program and indoor positioning is only available for large properties with more than 5 million annual visitors.
- Public building owners can choose to publish their indoor maps to the official Apple Maps.
- Apple provides an IMDF sandbox where IMDF maps can be validated and tested in a browser based interface. This sandbox is available for all Apple Developers and not only members of the Indoor Maps Program. See the demo of the sandbox at [15:30](https://developer.apple.com/wwdc19/245/?time=930).
- Indoor positioning:
  - Uses WiFi fingerprinting
  - Expected accuracy: 3-5 meters 
  - The indoor location on the indoor map can be obtained via CoreLocation.
  - To set up indoor positioning an indoor survey needs to be done. This is done by using the “Indoor Survey App” to collect WiFi info in the building, which is then uploaded for analysis and activation. The app is also used for testing the accuracy of the indoor positioning. See the demo of the app at [22:32](https://developer.apple.com/wwdc19/245/?time=1337).

- The content of an IMDF map:
  - Building Footprint
  - Levels
  - Units (room, walkaway, elevator etc.)
  - Openings (doors)
  - Kiosks (typically in the walkaway)
  - Labels and icons
  - Sections (highlights areas on the map eg. a food court in a mall)
