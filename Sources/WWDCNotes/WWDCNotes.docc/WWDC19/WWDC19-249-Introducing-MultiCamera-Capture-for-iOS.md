# Introducing Multi-Camera Capture for iOS

In AVCapture on iOS 13 it is now possible to simultaneously capture photos and video from multiple cameras on iPhone XS, iPhone XS Max, iPhone XR, and the latest iPad Pro. It is also possible to configure the multiple microphones on the device to shape the sound that is captured. Learn how to leverage these powerful capabilities to bring creative new features like picture-in-picture and spatial audio to your camera apps. Gain a deeper understanding of the performance considerations that may influence your app design.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/249", purpose: link, label: "Watch Video (44 min)")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- Supported on iPhone `XS, XS Max, XR, iPad Pro 3rd Gen`
- **AVCaptureMultiCamSession**
  - Multiple `AVCaptureDeiceInputs`, `AVCaptureDeiceOutputs` of the same type, `AVCaptureVideoPreviewLayers`
  - Don't use implicit connection forming but use `addInputWithNoConnections`, `addOutputWithNoConnections` or `AVCaptureVideoPreviewLayer.setSessionWithNoConnection`

- **AVCaptureSession** is still the way to got for single cam session
  - Simultaneous photo shooting, movie recording, barcode scanning, etc.

- **Limitiations**
  - only one input per camera in a session 
  - connecting one camera to multiple video data outouts is not possible
  - no presets supported on session since different cams might run with different qualities
  - multi-cam session has `hardwareCost` reporting. Session runnable when `0 <= cost <= 1` 
  - lower cost by `lower resolution`, `choose binned format`, `deviceInput.videoMinFrameDurationOverride = CMTimeMake(1, 30) to set max framerate override /* 30 FPS */`
  - lower system pressure like `temperature, power demands, infrared projector temperature` by 
    - lowering frame rates, throttle GPU/CPU processing code, disable one camera

- 
  - run indefinitely with `multiSession.systemPressureCost < 1.0`; device shutdown with `cost > 3.0`
  - iOS can run only one session at a time (with mutli cams though)

-  **Virtual Camera** is the new name wor software cameras like `True-Tone- or Dual-Camera`
  - `device.isVirtualDevice` - get its physical devices by `device.constituentDevices` for e.g. synchronized camera streaming
  - `AVCaptureDataOutputSynchronizer` ensures you get two outputs in one callback
  - virtual devices have **secret ports** so you can get 2 streams - you need to explicitly query them

- **Dual Camera Hography Aids**
  - `CameraIntrinsics` (Optical center / focal length) and `CameraExtrinsics` (rotation matrix / translation vector for both wide- and tele cameras)

- **Multi-Microphone capture**
  - By default front cam uses `front mic` and back cam uses `back mic`
  - actually `front mic` and `back mic` are a lie since different devices have multiple mics but not explicitly fron/back ones. This is achieved by `Microphone Beam Forming` - done automatically by CoreAudio `micInput.ports(for: .audio, sourceDeviceType: micDevice.deviceType, sourceDevicePosition: .front).first`
  - Beam forming only works with built-in mics
  - audio can be arbitrarily configured by creating custom `AVAudioSession`