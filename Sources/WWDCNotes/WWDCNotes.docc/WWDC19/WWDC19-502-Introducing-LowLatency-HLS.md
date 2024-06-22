# Introducing Low-Latency HLS

Since its introduction in 2009, HTTP Live Streaming (HLS) has enabled the delivery of countless live and on‐demand audio and video streams globally. With the introduction of a new Low-Latency mode, latencies of less than two seconds are now achievable over public networks at scale, while still offering backwards compatibility to existing clients. Learn about how to develop and configure your content delivery systems to take advantage of this new technology.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/502", purpose: link, label: "Watch Video (42 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- Crucial for `Sports`, `Late-breaking news`, `Game Streaming`, `Award ceremonies`, ...
- 2-8 seconds is **gold standard** and matches cable/satellite TV
- HLS targets **1-2 seconds** delay
- **Regular HLS** 
  - Simple and robust BUT comes with a cost (20-30s behind live)
  - This comes due to [client - server roundtrips](https://developer.apple.com/videos/play/wwdc2019/502/?time=225) and [too much caching] (https://developer.apple.com/videos/play/wwdc2019/502/?time=342)

- **Considerations**
  - HTTP is still king - but keeping it means keeping segment encode delay
  - CDNs are essentially to scale
  - Runway to react is much shorter

- **Changes**
  - **Reduce Publishing Latency** by allowing the server to publish small parts of the main segment before the main segment itself is ready
    - Leverage **Partial Segments**
    - Subset of regular segment - appear in parallel to them in playlist
    - Playlists update every partial segment
    - Publishing latency becomes the partial segment duration
    - Disappear quickly from playlist and are replaced/updated frequently

- 
  - **Optimize Segment Discovery** by allow asking for a particular playlist update in advance before it's actually ready on the server
    - Features enabled by **EXT-X-SERVER-CONTROL**
      - Small number of server directives 
      - Carried as query parameters on playlist URL (reserved in m3u8 URLs)
      - Sorted within URL to improve CDN hit ratio

- 
  - 
    - **Blocking Playlist Reload**
      - **EXT-X-SERVER-CONTROL** with **CAN-BLOCK-RELOAD** control
      - Clients ask for playlist update in advance
      - Server holds reuqest until next (partial) segment appears

- 
  - **Improve caching behavior** by allowing to have a different URL for each update
  - **Eleminate Segment Round Trip** by returning segments via push (HTTP/2) 
    - A playlist GET request also pushes new segment which eleminates additional rount trip

- 
  - **Reduce Playlist Transfer Overhead** by transmitting only delta updates if requested
    - Realized via `CAN-SKIP-UNTIL` control
    - Clients ask for delta updates explicitly
    - Update skips the earlier part of playlist

- 
  - **Switch Bitrate Tiers Quickly** by transmitting additional information with the updates
    - Playlist carry up-to-date reports on peer playlists
    - Last media sequence number and last partial segment number
    - Allows client to load latest playlist when switching bit rates

- **How Do Get Started?**
  - Deliver HLS via HTTP/2 and support **Push** + **Dependency and Weighting**
  - Each server must vend all bit rate tiers
  - CDN must aggregate duplicate pending requests to origin
  - Start implementing your origin - Spec for Low-Latency HLS is available now
  - During Beta period use App Entitlement `com.apple.developer.coremedia.hls.low-latency`
