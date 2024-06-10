# Use Xcode for server-side development

Discover how you can create, build, and deploy a Swift server app alongside your pre-existing Xcode projects within the same workspace. We'll show you how to create your own local app and test endpoints using Xcode, and explore how you can structure and share code between server and client apps to ease your development process

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/110360", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(Jeehut)
      @GitHubUser(zntfdr)
   }
}



## What building a server application in Swift looks like

Server applications are modeled as Swift packages, here's a simple manifest file:

```swift
// swift-tools-version: 5.7

import PackageDescription

let package = Package(
  name: "MyServer",
  platforms: [.macOS("12.0")],
  products: [
    .executable(name: "MyServer", targets: ["MyServer"]),
  ],
  dependencies: [
    // 👇🏻 Web framework, helps us structure our code and provides basic utilities like routing
    .package(url: "https://github.com/vapor/vapor.git", .upToNextMajor(from: "4.0.0")),
  ],
  targets: [
    // 👇🏻 This executable target maps the server application entry point
    .executableTarget(
      name: "MyServer",
      dependencies: [
        .product(name: "Vapor", package: "vapor")
      ]
    ),
    .testTarget(
      name: "MyServerTests",
      dependencies: ["MyServer"]),
  ]
)
```

Here's what our framework entry point might look like:

```swift
import Vapor

@main
public struct MyServer {
  public static func main() async throws {
    let webapp = Application()

    // 👇🏻 our server endpoints
    webapp.get("greet", use: Self.greet) 
    webapp.post("echo", use: Self.echo)

    // 👇🏻 server bootstrap code (provided by Vapor) 🚀
    try webapp.run()
  }

  static func greet(request: Request) async throws -> String {
    return "Hello from Swift Server"
  }

  static func echo(request: Request) async throws -> String {
    if let body = request.body.string {
      return body
    }
    return ""
  }
}
```

To run this server in our local machine via Xcode, we can:

1. select the <kbd>MyServer</kbd> scheme (generated for us by Xcode)
2. select the <kbd>My Mac</kbd> as the destination
3. hit <kbd>run</kbd>

Once the application has launched, we can use Xcode console to examine log messages emitted by the server. In the logs we can also see at which ip/port the server is listening (typically `127.0.0.1:8080`)

For this demo code you can test it via terminal

```shell
curl http://127.0.0.1:8080/greet; echo
curl http://127.0.0.1:8080/echo --data "Hello from WWDC 2022"; echo
```

## App side (Client side)

To communicate with the server we just created, we can use code similar to:

```swift
import Foundation

struct MyServerClient {
  let baseURL = URL(string: "http://127.0.0.1:8080")!

  func greet() async throws -> String {
    let url = baseURL.appendingPathComponent("greet")
    let (data, _) = try await URLSession.shared.data(for: URLRequest(url: url))
    guard let responseBody = String(data: data, encoding: .utf8) else {
      throw Errors.invalidResponseEncoding
    }
    return responseBody
  }

  enum Errors: Error {
    case invalidResponseEncoding
  }
}
```

## Deploying our server to the cloud

- There are many cloud providers to choose from: AWS, Google Cloud, Azure, Heroku, and many others
- most services offer a git push to deploy system
- to make it possible to just `git push` to deploy, cloud services use technologies like [buildpacks][buildpacks] to compile the application remotely and then deploy the binary artifacts to an ephemeral host

[buildpacks]: https://buildpacks.io 

## Storage options

Strategies:

- files for static data
  - good if application data is static or changes very slowly and manually

- iCloud
  - for user-centric data or global datasets (no need dedicated server)

- databases
  - for transactional data
  - the Swift open source community developed database drivers that help interact natively with most databases technologies:
    - [FoundationDB][fdb], [Redis][redis], Cassandra, [Postgres][fluent-postgres-driver], [DynamoDB][smoke-dynamodb], [MongoDB][fluent-mongo-driver], [SQLite][sql]

[fdb]: https://github.com/kirilltitov/FDBSwift
[sql]: https://github.com/vapor/fluent-sqlite-driver
[redis]: https://github.com/vapor/redis
[fluent-postgres-driver]: https://github.com/vapor/fluent-postgres-driver
[fluent-mongo-driver]: https://github.com/mongodb/mongo-swift-driver/
[smoke-dynamodb]: https://github.com/amzn/smoke-dynamodb
