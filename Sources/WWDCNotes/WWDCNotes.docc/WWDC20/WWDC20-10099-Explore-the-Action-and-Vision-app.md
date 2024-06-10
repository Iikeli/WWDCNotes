# Explore the Action & Vision app

It's now easy to create an app for fitness or sports coaching that takes advantage of machine learning&nbsp;—&nbsp;and to prove it, we built our own. Learn how we designed the Action & Vision app using Object Detection and Action Classification in Create ML along with the new Body Pose Estimation, Trajectory Detection, and Contour Detection features in the Vision framework. Explore how you can create an immersive application for gameplay or training from setup to analysis and feedback. And follow along in Xcode with a full sample project.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10099", purpose: link, label: "Watch Video (36 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## Theme
We can use our iPhone or iPad as a coach. The device can observe what we're doing and give us good, real-time feedback.

We already have the perfect tool in our pocket:

- High quality camera
- Fast CPU, GPU, and Neural Engine
- Comprehensive and cooperative APIs
- All on device, which means better privacy and no network latency

## Today in sports and fitness
- Sport analysis can help everyone improve
- Billions of enthusiasts and professional athletes who can benefit

We can go beyond watching a video to learn how to do an exercise. Common forms of analysis:

- What did the body do?
- Which objects are in motion? (Ex: Ball in soccer/tennis game)
- What is the field of play? (Ex: Soccer goal or tennis court)

The example in this session analyzes a simple-to-understand sport: bean bag toss. There's a simple application that can be downloaded.

## The "Action & Vision" App

![][bean_bag_rules]

Everyone wants to win! Some questions we may want answered:

- Why did I miss?
  - How was my bag flying?
  - What was my pose?
  - How fast was my throw?

- How do I keep score?
- Can I show off with different shots?

**For a demo, watch 3:40-5:03.**

![][key_algorithms]

- To find the boards, Apple trained a custom model and then used `VNCoreMLRequest`
- Scene stability: `VNTranslationalImageRegistrationRequest`
- Board measurements: `VNDetectContourRequest`
- Find player: `VNDetectHumanBodyPoseRequest` (new this year)
- Find throw trajectory: `VNDetectTrajectoriesRequest`
- Analyze throw type (overhand/underhand, etc): A new model Apple made that runs through `CoreML predict`.
- Speed measurements happen using existing board measurements and throw trajectory information.

### Detecting and recognizing boards
Apple trained a custom object detection model using the Create ML object detection template.

After training the model, Apple ran inference through Vision.

#### Why fixate the camera?
Some algorithms require a stable scene. Also, the playing field only needs to be analyzed once.

Another benefit is that, once the device is stabilized, we can assume the user wants to start analyzing. Therefore, there's no need for a "start" button. The user doesn't have to touch the screen at all.

### Scene stability through registration
`VNTranslationalImageRegistrationRequest` analyzes movement from one frame to the next. Once the pixel-difference between 2 frames is below some threshold, the device is considered stable.

### Finding the board
`VNDetectContourRequest` uses the bounding box we got from our object detection as a region of interest. These contours are then simplified for analysis.

To learn more about contour detection, see [Explore Computer Vision APIs](https://developer.apple.com/videos/play/wwdc2020/10673).

### Finding the player
`VNDetectHumanBodyPoseRequest` gives points corresponding to a person's body joints. We can then analyze joint angles using these points.

This data is also used for Action Classification.

To learn more, see [Detect Body and Hand Pose with Vision](https://developer.apple.com/videos/play/wwdc2020/10653).

## Building the App
*Most of the video from this point on is about the sample app's source code. To follow along, download the Xcode project from [this link](https://developer.apple.com/documentation/vision/building_a_feature-rich_app_for_sports_analysis). These notes attempt to capture the main take-aways from each portion of the source code discussed in the video.*

### How does the app progress through stages of the game?
The app uses a `GameManager` to manage its state and communicate changes to that state with its listening `ViewController`s. Here's how the `GameManager` singleton is initialized, directly from the sample project:

```swift
static var shared = GameManager()

private init() {
  // Possible states with valid next states.
  let states = [
    InactiveState([SetupCameraState.self]),
    SetupCameraState([DetectingBoardState.self]),
    DetectingBoardState([DetectedBoardState.self]),
    DetectedBoardState([DetectingPlayerState.self]),
    DetectingPlayerState([DetectedPlayerState.self]),
    DetectedPlayerState([TrackThrowsState.self]),
    TrackThrowsState([ThrowCompletedState.self, ShowSummaryState.self]),
    ThrowCompletedState([ShowSummaryState.self, TrackThrowsState.self]),
    ShowSummaryState([DetectingPlayerState.self])
  ]

  // Any state besides Inactive can be returned to Inactive.
  for state in states where !(state is InactiveState) {
      state.addValidNextState(InactiveState.self)
  }

  // Create state machine.
  stateMachine = GKStateMachine(states: states)
}
```

### `RootViewController`
*You can get an overview of the Storyboard by watching 10:25-10:58, but it doesn't seem necessary for the big picture. Essentially, `RootViewController` controls the rest of the app.*

This view controller is responsible for hosting the `CameraViewController` on `viewDidLoad()`. `CameraViewController` manages the frame buffers coming from either the camera or the selected video and passes it to `CameraViewControllerOutputDelegate`.

After setting up the input buffers, `RootViewController` calls `startObservingStateChanges()`, which registers the view controller to be notified by `GameManager` state changes. This is the second responsibility of `RootViewController`: presenting and dismissing overlaying view controllers based on the game state.

`RootViewController` has an extension where it conforms to the `GameStateChangeObserver` protocol and figures out which overlaying view controller to present. The options are:

- `SetupViewController`
- `GameViewController`
- `SummaryViewController`

Both `SetupViewController` and `GameViewController` also conform to `GameStateChangeObserver`. This means that they also listen for updates on game state and they become the current `CameraViewControllerOutputDelegate`.

### `SetupViewController`
In `viewDidAppear()`, this view controller creates a `VNCoreMLRequest` using the object detection model. This model was made using Create ML's Object Detection Transfer Learning algorithm in the Object Detection template.

### Detecting the board
As `SetupViewController` starts receiving buffers through `CameraViewControllerOutputDelegate` methods, it starts performing Vision requests on each buffer in the `detectBoard()` function. Here, the app takes object detection results and filters out results with low confidence. If it finds a result with sufficient confidence, it draws a bounding box on-screen around the detected object.

### Detecting board placement
In live camera mode, the app tells the user to align the bounding box with the board location guide, which is already present on the screen. Otherwise, in video playback mode, the app assumes that the board is placed on the right side of the scene.

### Detecting scene stability
The `checkSceneStability` function contains a `VNSequenceRequestHandler` because checking scene stability requires performing `VNTranslationalImageRegistrationRequest`s across a series of frames. It then adds the frame points to a `sceneStabilityHistoryPoints` array.

The view controller uses a read-only computed property `sceneStability` to calculate whether the scene is stable or not. This variable calculates the moving average of the points stored in `sceneStabilityHistoryPoints` and then calculates the distance. If the distance is less than 10 pixels, the scene is declared to be stable.

### Detecting contours
The `detectBoardContours()` function performs a `VNDetectContoursRequest` and sets the region of interest to the board's bounding box. This means the request will only look for contours in that region.

The `analyzeBoardContours()` function performs analysis of the top-level contours to find the board edge path and hole path.

## Transition
The `SetupViewController`'s responsibilities are now done. It has detected the board and its placement, ensured the scene is stable, and detected board contours.

Now, the app goes back to `RootViewController` with a new state `.DetectingPlayerState`. At this point, `RootViewController` presents `GameViewController`, which is presented as an overlay and becomes the new `CameraViewControllerOutputDelegate`.

## `GameViewController`
### Detecting Player
As `GameViewController` receives input buffers from the camera, it performs a `VNDetectHumanBodyPoseRequest`. The results are passed to the `humanBoundingBox(for:)` function. This function filters out low-confidence observations and returns the bounding box of the person who enters the frame.

### Detecting bean bag trajectories
The `VNDetectTrajectoriesRequest` finds moving objects while attempting to filter out noise movements. It does this by comparing differences in frames.

`VNDetectTrajectoriesRequest` is a *stateful* request, which means it builds state over time. This also means that we need to keep the request around, which is not mandatory for other requests in Vision.

A few frames down, once there's evidence of movement, a `VNTrajectoryObservation` is reported. However, we still know the time at which the trajectory started. This is contained in the `timeRange` property.

![][trajectory_detection]

Some more information on `VNTrajectoryObservation`:

![][trajectory_observation]

Notice how there's actually 2 parabolas above. The purple one is the bean bag, but the green one is the *shadow* of the bean bag.

#### Creating a `VNDetectTrajectoriesRequest`

```swift
request = VNDetectTrajectoriesRequest(
  frameAnalysisSpacing: CMTime.zero,  // 1
  trajectoryLength: trajectoryLength,  // 2
  completionHandler: { ... }  // 3
)

// 4
request.minimumObjectSize = beanBagRadiusNormalized
request.maximumObjectSize = beanBagRadiusNormalized + fudge
```

1. Setting the spacing to `.zero` means that we want to analyze all the frames, but we can change it to only start analyzing after a certain amount of time. This reduces computation cost.
2. Trajectory length is half the "width" of the parabola. Specifying this parameter helps with filtering out small movement noise that we don't care about.
3. The completion handler gives us a result, which we can use however we want.
4. Here, we specify size bounds for which object to track. This helps us avoid tracking the trajectory of the arm, for example, because it's too large. We can also filter out very small objects like a bug.

It's important to note that trajectory detection is happening on its own queue separate from the camera queue.

#### Things to keep in mind with trajectory detection

- Requires a stable scene
- Objects must travel on a parabola (lines are parabolas)
- Requires `CMSampleBuffer`s with timestamps
- Bounces create new trajectories; so does leaving the frame. To combine bounces into a single trajectory, we'll have to look at where end of trajectory 1 == start of trajectory 2
- Use region of interest if possible. This helps filter out noise movements.

#### Processing trajectory results
Going back to the code, `processTrajectoryObservations()` tracks information about each trajectory, such as its duration and points detected. It also updates the trajectory region of interest if the trajectory is still in flight. If the trajectory is outside the region of interest for more than 20 frames, the app considers the throw to be complete.

It's important to note that `trajectoryView.points` has a `didSet` observer which calls `updatePathLayer()`. This function updates the trajectory path in the view. It also checks if the trajectory points are within the region of interest. Lastly, this function calculates the release speed of the throw.

### Detecting type of throw
Apple created a custom classification model using the Create ML Action Classification template. They collected video of throw types that they wanted to classify, but they also collected video of non-throwing actions like walking.

Action Classification uses Bodypose through Vision. Just like trajectory detection, this builds evidence over time.

![][action_classification]

To determine the last throw a player made, we can go to the `cameraViewController()` delegate method inside `GameViewController`. For each buffer received, the method is also performing a `VNDetectHumanBodyPoseRequest` and storing those points as observations. Those observations are using in the `humanBoundingBox(for:)` function and used as input to the Action Classification model.

Once a throw is complete, the player's stats are updated. At this point, `playerStats.getLastThrowType()` is called. Here's the implementation:

```swift
struct PlayerStats {
  mutating func getLastThrowType() -> ThrowType {
      guard
        let actionClassifier = try? PlayerActionClassifier(configuration: MLModelConfiguration()),
        let poseMultiArray = prepareInputWithObservations(poseObservations),  // 1
        let predictions = try? actionClassifier.prediction(poses: poseMultiArray),  // 2
        let throwType = ThrowType(rawValue: predictions.label.capitalized)
      else {
          return .none
      }

      return throwType
  }
}
```

1. Stored `poseObservations` are used as the input to the classifier. `prepareInputWithObservations()` is a helper function that massages data into a form required by the model.
2. The throw type with the highest probability is returned.

### Metrics calculated on trajectories
We want to calculate the size of the field. We know the physical size of the board (4' x 2'). We measure the contour of the board. From this data, we can calculate a mapping of pixel measurements to the real world.

We also know that the trajectory is in the same plane as the ball. So, we can calculate the speed using the size of the real world.

We can also calculate the release angle using:

- Body pose at the beginning of the throw
- Angle between relevant points (elbow/wrist vs horizontal plane)

In code, the `updatePlayerStats()` function gets the release speed, trajectory, and release angle of the throw. When the app first observes a trajectory, it calculates the length of that trajectory in pixels. This can be converted to a real-world distance using the game board length as a reference.

From here, release speed is simply the length of the trajectory divided by the duration of the trajectory.

The release angle is calculated in the `getReleaseAngle` function, which uses the angle between the forearm and the horizontal plane:

```swift
mutating func getReleaseAngle() -> Double {
  if !poseObservations.isEmpty {
    let observationCount = poseObservations.count
    let postReleaseObservationCount = GameConstants.trajectoryLength + GameConstants.maxTrajectoryInFlightPoseObservations
    let keyFrameForReleaseAngle = observationCount > postReleaseObservationCount ? observationCount - postReleaseObservationCount : 0
    let observation = poseObservations[keyFrameForReleaseAngle]
    let (rightElbow, rightWrist) = armJoints(for: observation)
    // Release angle is computed by measuring the angle forearm (elbow to wrist) makes with the horizontal
    releaseAngle = rightElbow.angleFromHorizontal(to: rightWrist)
  }
  return releaseAngle
}
```

## Important points when creating both ML models
### Object Detection
Model input data should be as close to real-world usage as possible. The input data for object detection was:

- Captured with iPhone
- Standard bean bag boards
- Images of boards outside because that's where games are usually played.

Apple realized they forgot to include actual images of bean bags or people, so the model initially had difficulty detecting boards when people and bean bags were in the frame. To fix this, they added pictures with people and bean bags in the frame. They also took pictures from various distances and angles.

![][input_data_object_detection]

### Action Classification
Similarly, Action Classification data was captured with iPhone at various distances and angles.

#### Roadblock 1
Initially, throw classifications were just "underhand", "overhand", and "under the leg". However, all actions, such as picking up a bean bag, were recognized as one of those 3 actions.

To account for these additional situations, Apple added another Negative class of actions, where people were not throwing bean bags. Doing this helped the model's performance.

![][negative_actions]

#### Roadblock 2
Another issue was figuring out the correct prediction window. It needed to be set to capture the *entire* target action. Apple settled at 45 frames.

A final issue was figuring out how frequently to run prediction. For this app, frames were sent once a throw was completed.

## Best practices for live processing
### Challenges of live streams

- Camera only has finite buffers
  - Buffers we hold are not available to the camera. If we don't release them ASAP, we can starve the camera.
- Assuming our algorithm is always faster than the throw is a bad assumption because load on the system varies.

We can use the following delegate method from `AVCaptureVideoDataOutputSampleBufferDelegate` to be notified when we drop some buffers:

```swift
func captureOutput(
  _ output: AVCaptureOutput,
  didDrop sampleBuffer: CMSampleBuffer,
  from connection: AVCaptureConnection
)
```

### Dealing with live streams

- Speed is important, and there's a lot of things to analyze. Multiple tasks should run in parallel on different queues.
- Rendering should not happen at the end, because that must happen on the main queue and will bottleneck our app. 
  - Instead, we should release the buffer and then asynchronously render on the main queue.

### Dealing with live playback
Live video playback has challenges similar to live streams:

- Don't go frame by frame
- Use `AVPlayerItemVideoOutput` and a `CADisplayLink`.

Here's some code from the project (type-annotated for clarity):

```swift
let nextTimeStamp: CFTimeStamp = displayLink.timestamp + displayLink.duration
let itemTime: CMTime = output.itemTime(forHostTime: nextTimeStamp)
guard output.hasNewPixelBuffer(forItemTime: itemTime) else { return }
guard let pixelBuffer = output.copyPixelBuffer(forItemTime: itemTime, itemTimeForDisplay: nil) else { return }

// ... do stuff with buffer
```

## Wrap up
This type of application can work for other sports, like tennis, soccer, and cricket (35:40 - 36:00).

[bean_bag_rules]: WWDC20-10099-bean_bag_rules

[key_algorithms]: WWDC20-10099-key_algorithms

[trajectory_detection]: WWDC20-10099-trajectory_detection

[trajectory_observation]: WWDC20-10099-trajectory_observation

[action_classification]: WWDC20-10099-action_classification

[input_data_object_detection]: WWDC20-10099-input_data_object_detection

[negative_actions]: WWDC20-10099-negative_actions
