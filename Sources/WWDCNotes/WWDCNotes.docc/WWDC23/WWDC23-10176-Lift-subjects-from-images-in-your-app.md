# Lift subjects from images in your app

Discover how you can easily pull the subject of an image from its background in your apps. Learn how to lift the primary subject or to access the subject at a given point with VisionKit. We’ll also share how you can lift subjects using Vision and combine that with lower-level frameworks like Core Image to create fun image effects and more complex compositing pipelines.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10176", purpose: link, label: "Watch Video (18 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speakers: Lizzy Board and Saumitro Dasgupta

## What is a subject?

> A subject is the foreground object, or objects, of a photo. This is not always a person or a pet. It can be anything from a building, a plate of food, or some pairs of shoes.

## Supporting frameworks

### VisionKit

- Allows you to very easily adopt system-like subject lifting behavior, right out of the box.
- Lets you easily recreate the subject lifting UI that we all know and love, with just a few lines of code.
- Exposes some basic information about these subjects.
- All happens out-of-process, which has performance benefits but mens the image size is limited.

### Vision

- Lower-level framework that doesn't have out-of-the-box UI.
- Support multiple input sources and higher image resolution.
- Good for more advanced image editing pipelines.

## Subject lifting in VisionKit

**iOS**:

1. Initialize an `ImageAnalysisInteraction`.
2. Add the interaction to a view containing the image (this can be an `UIImageView`, but doesn't need to be).

**macOS**:

1. Initialize an `ImageAnalysisOverlayView`.
2. Add the view as a subview of the `NSView` containing the image.

A preferred interaction type can be set on the `ImageAnalysisInteraction` and `ImageAnalysisOverlayView` to choose which interactions should be supported.

- Default is `.automatic` which mirrors system behavior. This supports subject lifting, live text and data detectors.
- The new `.imageSubject` supports only subject lifting.

### Manually analyzing images

```swift
let analyzer = ImageAnalyzer()
let analysis = try? await analyzer.analyze(image, configuration: configuration)
```

The `ImageAnalysis` struct has a property `subjects` which contains a list of the image's `Subject`s which contains an image and its bounds. It also has a property `highlightedSubjects` which contains the highlighted subjects. This can be changed programatically.

### Looking up a subject

```swift
let subject = try? await interaction.subject(at: point)
```

If there are no subjects at that point, this method will return nil.

### Generating subject images

```swift
// For an image for a single Subject:
subject.image
// For an image composed of multiple Subjects:
interaction.image(for: interaction.subjects)
```

## Subject lifting in Vision

### Different kind of APIs

**Saliency:**

- VNGenerateAttentionBasedSaliencyImageRequest
- VNGenerateObjectnessBasedSaliencyImageRequest

> Saliency requests, like the ones for attention and objectness, are best used for coarse, region-based analysis.

> The generated saliency maps are at a fairly low resolution and as such, not suitable for segmentation. Instead, you could use the salient regions for tasks like auto-cropping an image.

**Person segmentation:**

- VNGeneratePersonSegmentationRequest

> It shines at producing detailed segmentation masks for people in the scene. Use this if you specifically want to focus on segmenting people.

**Person instance segmentation:**

- VNGeneratePersonInstanceMaskRequest _**NEW**_

> The new person instance segmentation API takes things further by providing a separate mask for each person in the scene.

*See more in the session "Explore 3D body pose and person segmentation in Vision".*

**Subject lifting:**

- VNGenerateForegroundInstanceMaskRequest _**NEW**_

> The newly introduced subject lifting API is "class agnostic". Any foreground object, regardless of its semantic class, can be potentially segmented.

### Concepts

> You start with an input image. The subject lifting request processes this image and produces a soft segmentation mask at the same resolution. Taking this mask and applying it to the source image results in the masked image.

> Each distinct segmented object is referred to as an instance.

> Vision also provides you with pixelwise information about these instances. This instance mask maps pixels in the source image to their instance index. The zero index is reserved for the background, and then each foreground instance is labeled sequentially, starting at 1.

The ordering of the IDs are not guaranteed.

### Generate a masked image

```swift
let request = VNGenerateForegroundInstanceMaskRequest()
let handler = VNImageRequestHandler(cgImage: image)
try handler.perform([request])
guard let result = request.results?.first else {
  return
}
let output = result.generateMaskedImage(
  ofInstances: result.allInstances,
  from: requestHandler,
  croppedToInstancesExtent: false)
```

This is a resource intensive task and best deferred to a background thread so as not to block the UI.

### Working with masks

```swift
result.generateScaledMaskForImage(
  forInstances: result.allInstances,
  from: requestHandler)
```

> The mask I just generated is perfectly suited for use with CoreImage. Vision, much like VisionKit, produces SDR outputs. Performing the masking in CoreImage, however, preserves the high dynamic range of the input.

```swift
func apply(mask: CIImage, toImage image: CIImage) -> CIImage {
  let filter = CIFilter.blendWithMask()
  filter.inputImage = image
  filter.maskImage = mask
  filter.backgroundImage = CIImage.empty()
  return filter.outputImage!
}
```

*See more in the session "Support HDR images in your app".*

### Demo app / putting it all together

Between 13:22 and 18:04, the code snippets above, and some more are put together, to enable a visual effects demo app.
