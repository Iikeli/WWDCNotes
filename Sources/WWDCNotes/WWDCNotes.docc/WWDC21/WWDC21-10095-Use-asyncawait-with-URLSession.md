# Use async/await with URLSession

Discover how you can adopt Swift concurrency in URLSession using async/await and AsyncSequence, and how you can apply Swift concurrency concepts to improve your networking code.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10095", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(donnywals)
      @GitHubUser(zntfdr)
   }
}



## Fetching data

Two main methods:

```swift
func data(from url: URL) async throws -> (Data, URLResponse)
func data(for request: URLRequest) async throws -> (Data, URLResponse)
```

Example:

```swift
// Fetch photo with async/await

func fetchPhoto(url: URL) async throws -> UIImage {
  let (data, response) = try await URLSession.shared.data(from: url)

  guard let httpResponse = response as? HTTPURLResponse,
        httpResponse.statusCode == 200 else {
    throw MyNetworkingError.invalidServerResponse
  }

  guard let image = UIImage(data: data) else {
    throw MyNetworkingError.unsupportedImage
  }

  return image
}
```

## Upload data

Two main methods:

```swift
func upload(for request: URLRequest, from data: Data) async throws -> (Data, URLResponse)
func upload(for request: URLRequest, fromFile url: URL) async throws -> (Data, URLResponse)
```

Example:

```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"

let (data, response) = try await URLSession.shared.upload(for: request, fromFile: fileURL)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 201 /* Created */ else {
  throw MyNetworkingError.invalidServerResponse
}
```

## Download data

Methods:

```swift
func download(from url: URL) async throws -> (URL, URLResponse)
func download(for request: URLRequest) async throws -> (URL, URLResponse)
func download(resumeFrom resumeData: Data) async throws -> (URL, URLResponse)
```

Unlike download task convenience methods, these new methods do not automatically delete the file: don't forget to do so yourself.

Example:

```swift
let (location, response) = try await URLSession.shared.download(from: url)
guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 /* OK */ else {
  throw MyNetworkingError.invalidServerResponse
}

try FileManager.default.moveItem(at: location, to: newLocation)
```

## Cancellation

Swift concurrency's cancellation works with URLSession async methods: calling `cancel()` on a task that contains running network operations will cancel such operations.

```swift
let task = Task {
  let (data1, response1) = try await URLSession.shared.data(from: url1)
  let (data2, response2) = try await URLSession.shared.data(from: url2)
}

task.cancel()
```

## Increment download `URLSession.bytes`

URLSession.bytes methods return when the response headers have been received and deliver the response body as an AsyncSequence of bytes.

Example:

```swift
let (bytes, response) = try await URLSession.shared.bytes(from: Self.eventStreamURL)

guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
  throw WoofError.invalidServerResponse
}

for try await line in bytes.lines {
  let photoMetadata = try JSONDecoder().decode(PhotoMetadata.self, from: Data(line.utf8))
  await updateFavoriteCount(with: photoMetadata) // 👈🏻 need to execute in the main actor
}
```

## URLSessionTask-specific delegate

`URLSession` is designed around a delegate model which provides callbacks for events such as authentication challenges, metrics, and more. 

The new async methods no longer expose the underlying `URLSession.task`, instead, we specify a delegate via an optional argument, a task-specific delegate, allowing you to provide an object to handle delegate messages specific to this data upload, download, or bytes operation.

```swift
func data(from url: URL, delegate: URLSessionTaskDelegate?)
func data(for request: URLRequest, delegate: URLSessionTaskDelegate?)
func upload(for request: URLRequest, fromFile url: URL, delegate: URLSessionTaskDelegate?)
func upload(for request: URLRequest, from data: Data, delegate: URLSessionTaskDelegate?)
func download(from url: URL, delegate: URLSessionTaskDelegate?)
func download(for request: URLRequest, delegate: URLSessionTaskDelegate?)
func download(resumeFrom resumeData: Data, delegate: URLSessionTaskDelegate?)
func bytes(from url: URL, delegate: URLSessionTaskDelegate?)
func bytes(for request: URLRequest, delegate: URLSessionTaskDelegate?)
```

- the same task is supported in Objective-C
- the task is strongly held by a task until it completes or fails
- task-specific delegate is not supported by background URLSession
- If a method is implemented on both session delegate and task delegate, the one on task delegate will be called

Example:

```swift
class AuthenticationDelegate: NSObject, URLSessionTaskDelegate {
  private let signInController: SignInController
  
  init(signInController: SignInController) {
    self.signInController = signInController
  }
  
  func urlSession(
  	_ session: URLSession,
    task: URLSessionTask,
    didReceive challenge: URLAuthenticationChallenge
  ) async -> (URLSession.AuthChallengeDisposition, URLCredential?) {
    if challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodHTTPBasic {
      do {
        let (username, password) = try await signInController.promptForCredential()
        return (.useCredential, URLCredential(user: username, password: password, persistence: .forSession))
      } catch {
        return (.cancelAuthenticationChallenge, nil)
      }
    } else {
      return (.performDefaultHandling, nil)
    }
  }
}

...

let (bytes, response) = try await URLSession.shared.bytes(
	from: Self.eventStreamURL, 
	delegate: AuthenticationDelegate()
)
```
