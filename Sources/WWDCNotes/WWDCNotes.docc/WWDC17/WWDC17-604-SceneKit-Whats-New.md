# SceneKit: What's New

SceneKit is a fast and fully featured high-level 3D graphics framework that enables your apps and games to create immersive scenes and effects. See the latest advances in camera control and effects for simulating real camera optics including bokeh and motion blur. Learn about surface subdivision and tessellation to create smooth-looking surfaces right on the GPU starting from a coarser mesh. Check out new integration with ARKit and workflow improvements enabled by the Xcode Scene Editor.

@Metadata {
   @TitleHeading("WWDC17")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc17/604", purpose: link, label: "Watch Video (53 min)")

   @Contributors {
      @GitHubUser(thecodedself)
   }
}



## Camera enhancements

Cameras follow objects with a smooth acceleration and deceleration. They also can adapt their behavior in different areas, such as moving up or down depending on the area, or remaining fixed in a certain orientation.

SceneKit is transitioning to a physically based camera API. This API allows implementation of physically plausible depths of field, as well as motion blur and screen space ambient occlusion. 

### New API

Rather than specifying `xFov` and `yFov` properties, now configure the `fieldOfView` in degrees or specify the `focalLength` and `sensorHeight` of the camera (analogous to a physical camera).

Setting `fieldOfView` adjusts `focalLength` and `sensorHeight` and vice versa.

You can also configure depth of field with `wantsDepthOfField`, `focusDistance` and `fStop`.

Depth of field also comes with automatic bokeh. Use of an HDR camera is recommended, done by setting `wantsHDR = true`. Configure the bokeh with `apertureBladeCount`.

Set motion blur with `motionBlurIntensity`. Works per object.

**Ambient occlusion** affects how light reflects off of objects with depth where not all parts of the surface get the same amount of light. You'll see shading and shadows on the object. Set by the `screenSpaceAmbientOcclusion...` family of properties.

### Camera control

`SCNCameraController` allows you to manipulate the camera. Usually accessed with `SCNView.defaultCameraController`, but you can instantiate your own.

Use an `SCNCameraController` to allow users to manipulate the camera with tap gestures. There are different camera modes. Some focus on an object and allow rotation, others allow you to fly throughout the scene, and more.

#### Camera behavior

Use `SCNConstraints` to define camera behavior if you need something more complicated for certain games or apps. Some examples of constraints:

- `SCNLookAtConstraint` will keep focus on an object as it moves.
- `SCNReplicatorConstraint` replicates the behavior of an object. For example, the default camera behavior, it'll follow the object.
- `SCNDistanceConstraint` will keep you within a min/max distance of an object.

## Tessellation and subdivision surfaces

**Tessellation** lets you provide the GPU with low-resolution models which then generates models in memory that are much higher detail. SceneKit does tessellation with `SCNGeometryTessellator`.

### Shader modifiers

Shader modifiers create custom effects during tessellation. You can use it to create waves in a body of water, for example.

### Displacement mapping

Modifies a surface.

Height maps modify a surface with changes in the height of its vertexes. Imagine starting with a perfectly smooth surface representing land on an alien planet and then applying a height map to add hills and valleys.

Vector displacement maps are an extension of height maps that also let you modify vectors in all three dimensions. Create a 3D rock from a flat surface and add texture, for example

### Subdivision surfaces

You can use SceneKit to start with a coarse model and refine it smoother. Like, a cube into a sphere. But not objects are uniformly smooth. Subdivision surfaces can create different angles of the surfaces in different parts of the model.

This has been available in SceneKit but is now moving to the GPU for increased performance.

Feature-adaptive subdivision is now supported. Using creases, you can make certain parts of a model smooth, and others more blocky.

Keep two things in mind if you'll be using subdivision surfaces:

1. When loading `SCNScene` from files, make sure to specify the `.preserveOriginalTopology: true` option.
2. When creating `SCNGeometryElement` objects programmatically, use the `.polygon` primitive type. This is to use quads, not triangles.

## Animation improvements

New `SCNAnimation` protocol and `SCNAnimationPlayer` class that make it easy to start animations and mutate them live.

You can blend animations together, too.

## Developer tools

A new SceneKit Instrument helps with:

- Understanding performance issues
- Recording a trace of SceneKit's behavior
- Providing accurate per-frame performance analysis

SceneKit's Scene Editor also has new features, including a new Shader Modifier Editor.

## Related technologies

### ARKit

Support for ARKit using `ARSCNView`, a subclass of `SCNView`. Very easy to set a texture or SceneView's background to the output of an `AVCaptureDevice`.

You can have objects in an AR scene cast shadows too.

### GameplayKit

GameplayKit components can drive SceneKit objects. 

### Model I/O

- Improved support for USD
- Better material bridging
- Support for animations

### UIFocus

`SCNNode` conforms to `UIFocusItem` to let you select and focus on objects on the Apple TV using your remote.

## Rendering Additions

- Support for point cloud rendering
- New transparency modes
- Support for cascaded shadow maps
