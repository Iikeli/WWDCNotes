# Accelerate networking with HTTP/3 and QUIC

The web is changing, and the next major version of HTTP is here. Learn how HTTP/3 reduces latency and improves reliability for your app and discover how its underlying transport, QUIC, unlocks new innovations in your own custom protocols using new transport functionality and multi-streaming connection groups.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10094", purpose: link, label: "Watch Video (19 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Evolution of HTTP

- HTTP/1.1 
  - initially we had to make a new connection for every resource that we wanted from the server
  - when doing so, a lot of time is spent on connection setup instead of resource transmission
  - we reuse a single HTTP/1 connection for multiple data transfers, a.k.a. [head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking), but a request can only be sent after the previous response has ended
  - to over come this, in the past, HTTP implementations used many parallel connections, however this brings inefficient networking behaviors for both client and server

- HTTP/2 solves head-of-line blocking by multiplexing multiple streams on a single connection
  - requests are sent earlier, and data from different streams can be interleaved
  - this allows more efficient use of a single TCP connection, as idle waiting time is drastically reduced

- HTTP/3
  - connections are set up much faster
  - streams are independent (in HTTP/2, all streams shared a single TCP connection) - packet loss only affect one stream but not others
  - uses [QUIC](https://en.wikipedia.org/wiki/QUIC) instead of TCP

### QUIC

- new transport protocol 
- faster connection setup
- connection migration support - allows connections to move seamlessly across different network interfaces without reestablishing a session
- [TLS 1.3](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.3) security
- standardized by the [Internet Engineering Task Force (IETF)](https://www.ietf.org)
- based on the same concepts of TCP
- provides: 
  - end-to-end encryption
  - multiplexed streams
  - authentication

## Using HTTP/3

- enabled by default in `URLSession` - you only need to enable HTTP/3 on your server
- supports HTTP/3 RFC and Draft 29

You can use Instruments to verify that your app uses HTTP/3 - use the <kbd>networking profiling template</kbd> to inspect HTTP Traffic

## Using QUIC

When to use QUIC directly:

- Non-request/response pair
- Benefits from multiplexed streams
- Custom protocols

Using QUIC in your app:

- new [`NWProtocolQUIC`][NWProtocolQUIC] built-in in Network.framework

```swift
// Create a connection using QUIC
let connection = NWConnection(
  host: "example.com", 
  port: 443, 
  using: .quic(alpn: ["myproto"]) // 👈🏻
)

// Set the state update handler to be notified when the connection is ready
connection.stateUpdateHandler = { newState in
  switch newState {
  case .ready:
    print("Connected using QUIC!")
  default:
    break
  }
}

// Start the connection with callback queue
connection.start(queue: queue)
```

Using QUIC streams to send and receive data

```swift
// Send data on a stream
connection.send(content: data, completion: .contentProcessed { error in
  // Handle error, if any
  // Schedule next send
})

// Receive incoming data
connection.receive(minimumIncompleteLength: 1, maximumLength: desiredLength) { 
  receivedcontent, context, isComplete, receivedError in
  // Handle data, error
  // Schedule next receive
}
```

Use [`NWMultiplexGroup`][NWMultiplexGroup] to refer to the underlying transport shared by the group of streams.

- A Connection Group follows a lifecycle similar to that of the other Network.framework objects and allows you to reason about the state of the underlying QUIC tunnel shared by your QUIC streams
- It also allows you to create new outgoing streams from a specific QUIC tunnel as well as receive new incoming streams initiated by the remote endpoint

Establish a tunnel with NWMultiplexGroup:

```swift
// Create a group
let descriptor = NWMultiplexGroup(to: .hostPort(host: "example.com", port: 443))
let group = NWConnectionGroup(with: descriptor, using: .quic(alpn: ["myproto"]))

// Set the state update handler to be notified when the group is ready
group.stateUpdateHandler = { newState in
  switch newState {
  case .ready:
    print("Connected using QUIC!")
  default:
    break
  }
}

// Start the group with callback queue
group.start(queue: queue)
```

Manage streams with `NWConnectionGroup`:

```swift
// Create a new outgoing stream
let connection = NWConnection(from: group)

// Receive new incoming streams initiated by the remote endpoint
group.newConnectionHandler = { newConnection in

  // Set state update handler on incoming stream
  newConnection.stateUpdateHandler = { newState in
    // Handle stream states
  }

  // Start the incoming stream
  newConnection.start(queue: queue)

}
```

Receive incoming QUIC tunnels from `NWListener`:

```swift
// Set the new connection group handler
listener.newConnectionGroupHandler = { group in

  group.stateUpdateHandler = { newState in
    // Handle tunnel states
  }

  group.newConnectionHandler = { stream in
    // Set up and start new incoming streams
  }

  group.start(queue: queue)

}
```

Access QUIC metadata to learn about and modify streams:

```swift
// Find the stream ID of a particular QUIC stream
if let metadata = connection.metadata(definition: NWProtocolQUIC.definition)
               as? NWProtocolQUIC.Metadata {
  print("QUIC Stream ID is \(metadata.streamIdentifier)")

  // Some time later...

  // Set the application error, if appropriate, before cancelling the stream
  metadata.applicationError = 0x100
  connection.cancel()
}
```

[NWProtocolQUIC]: https://developer.apple.com/documentation/network/nwprotocolquic
[NWMultiplexGroup]: https://developer.apple.com/documentation/network/nwmultiplexgroup