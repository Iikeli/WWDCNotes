# What’s new in voice processing

Learn how to use the Apple voice processing APIs to achieve the best possible audio experience in your VoIP apps. We’ll show you how to detect when someone is talking while muted, adjust ducking behavior of other audio, and more.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10235", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Julian Yu, Core Audio

## Voice Processing APIs

Both APIs provides the same voice processing capabilities.

**NEW:** Voice processing now available on tvOS

*See more in the session "Discover Continuity Camera on tvOS".*

### AUVoiceIO (aka AUVoiceProcessingIO)

For apps that interact with I/O audio unit directly

### AVAudioEngine

Higher-level API, easier to use, less code

## New APIs

AUVoiceIO and AVAudioEngine get new API to provide more controls over voice processing.

### Ducking behavior

> There also could be other apps playing audio at the same time as your app. All the audio streams other than the voice audio stream from your app are considered as "other audio" by Apple's voice processing, and your voice audio gets mixed in with other audio before getting played to the output device. For voice chat apps, typically the primary focus of the playback audio is the voice chat audio. That's why we duck the volume level of other audio, in order improve the intelligibility of the voice audio. In the past, we applied a fixed amount of ducking to other audio. This has worked well for most apps, and if your app is happy with the current ducking behavior, then you don't need to do anything.

When enable advanced ducking a ducking level can be set. There is default (the current behavior), minimum, medium and maximum, where maximum lowers the other apps playing audio the most.

#### Enabling it with AUVoiceIO

```swift
const AUVoiceIOOtherAudioDuckingConfiguration duckingConfig = {
	.mEnableAdvancedDucking = true,
	.mDuckingLevel = AUVoiceIOOtherAudioDuckingLevel::kAUVoiceIOOtherAudioDuckingLevelMin
};
// AUVoiceIO creation code omitted
OSStatus err = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_OtherAudioDuckingConfiguration, kAudioUnitScope_Global, 0, &duckingConfig, sizeof(duckingConfig));
```

#### Enabling it with AVAudioEngine

```swift
let engine = AVAudioEngine()
let inputNode = engine.inputNode
do {
	try inputNode.setVoiceProcessingEnabled(true)
} catch {
	print("Could not enable voice processing \(error)")
}
let duckingConfig = AVAudioVoiceProcessingOtherAudioDuckingConfiguration(mEnableAdvancedDucking: false, mDuckingLevel: .max)
inputNode.voiceProcessingOtherAudioDuckingConfiguration = duckingConfig
```

### Detect muted speakers

There is an API to detect presence of a muted talker

It was first introduced in iOS 15, and now available in macOS 14 and tvOS 17.

#### How to use the API

1. Provide a listener block to receive the notification of muted talker detection
2. Implement your handling of this notification in your listener block
3. Implement mute via the mute API

#### Using it with AUVoiceIO

```swift
UVoiceIOMutedSpeechActivityEventListener listener =  ^(AUVoiceIOMutedSpeechActivityEvent event) {		
    if (event == kAUVoiceIOSpeechActivityHasStarted) {
		// User has started talking while muted. Prompt the user to un-mute
	} else if (event == kAUVoiceIOSpeechActivityHasEnded) {
		// User has stopped talking while muted
	}
};
OSStatus err = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_MutedSpeechActivityEventListener, kAudioUnitScope_Global, 0, &listener,  sizeof(AUVoiceIOMutedSpeechActivityEventListener));
// When user mutes
UInt32 muteUplinkOutput = 1;
result = AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_MuteOutput, kAudioUnitScope_Global, 0, &muteUplinkOutput, sizeof(muteUplinkOutput));
```

#### Using it with AVAudioEngine

```swift
let listener =  { (event : AVAudioVoiceProcessingSpeechActivityEvent) in
	if (event == AVAudioVoiceProcessingSpeechActivityEvent.started) {
		// User has started talking while muted. Prompt the user to un-mute
	} else if (event == AVAudioVoiceProcessingSpeechActivityEvent.ended) {
		// User has stopped talking while muted
	}
}
inputNode.setMutedSpeechActivityEventListener(listener)
// When user mutes
inputNode.isVoiceProcessingInputMuted = true
```

#### Alternative macOS APIs (if not ready for the new one)

1. Enable voice activity detection on input device via `kAudioDevicePropertyVoiceActivityDetectionEnable`.
2. Register a listener callback on HAL property `kAudioDevicePropertyVoiceActivityDetectionState`.
3. When notified, query `kAudioDevicePropertyVoiceActivityDetectionState` for its current value

```swift
// Enable Voice Activity Detection on the input device
const AudioObjectPropertyAddress kVoiceActivityDetectionEnable{
        kAudioDevicePropertyVoiceActivityDetectionEnable,
        kAudioDevicePropertyScopeInput,
        kAudioObjectPropertyElementMain };
OSStatus status = kAudioHardwareNoError;
UInt32 shouldEnable = 1;
status = AudioObjectSetPropertyData(deviceID, &kVoiceActivityDetectionEnable, 0, NULL, sizeof(UInt32), &shouldEnable);
// Register a listener on the Voice Activity Detection State property
const AudioObjectPropertyAddress kVoiceActivityDetectionState{
        kAudioDevicePropertyVoiceActivityDetectionState,
        kAudioDevicePropertyScopeInput,
        kAudioObjectPropertyElementMain };
status = AudioObjectAddPropertyListener(deviceID, &kVoiceActivityDetectionState, (AudioObjectPropertyListenerProc)listener_callback, NULL); // “listener_callback” is the name of your listener function
```

```swift
OSStatus listener_callback(
    AudioObjectID                 inObjectID,
    UInt32                        inNumberAddresses,
    const AudioObjectPropertyAddress*   __nullable inAddresses,
    void* __nullable              inClientData)
{
  // Assuming this is the only property we are listening for, therefore no need to go through inAddresses
       UInt32 voiceDetected = 0;
     UInt32 propertySize = sizeof(UInt32);
     OSStatus status = AudioObjectGetPropertyData(inObjectID, &kVoiceActivityState, 0, NULL, &propertySize, &voiceDetected);
  
       if (kAudioHardwareNoError == status) {
 if (voiceDetected == 1) {
    // voice activity detected
	} else if (voiceDetected == 0) {
		    // voice activity not detected
	}
 }
 return status;
};
```

> For HAL API clients to implement mute we highly recommend using HAL's process mute API. It suppresses the recording indicator light in the menu bar, and gives user confidence that their privacy is protected under mute. 
