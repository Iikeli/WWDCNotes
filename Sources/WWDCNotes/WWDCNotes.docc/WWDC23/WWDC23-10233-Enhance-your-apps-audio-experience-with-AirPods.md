# Enhance your app’s audio experience with AirPods

Discover how you can create transformative audio experiences in your app using AirPods. Learn how to incorporate AirPods Automatic Switching, use AVAudioApplication to support Mute Control, and take advantage of Spatial Audio to create immersive soundscapes in your app or game.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10233", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(multitudes)
   }
}



## Chapters:
[04:20](https://developer.apple.com/wwdc23/10233?time=260) AirPods Automatic Switching for macOS  
[06:41](https://developer.apple.com/wwdc23/10233?time=401) Press to Mute and Unmute  
[12:02](https://developer.apple.com/wwdc23/10233?time=722) Spatial Audio with AirPods

## Intro
In this session, we will explore how you can enhance your app's audio experience with AirPods. We will introduce you to some of the AirPods features we announced for iOS 17 and macOS 14.

## One of the most common use cases
You are enjoying music on AirPods in Personalized Spatial Audio with iPhone and receive a notification to join a work meeting. As you unlock your Mac to join the call, you are greeted by an AirPods Connected banner, letting you know that the AirPods are now ready and connected to the Mac.

![connected banner][connected]  

[connected]: connected.jpg

Next, as you join the meeting, audio will be played from Mac through your AirPods as you would expect, and the music you were listening to on iPhone pauses. In the middle of your work call, someone in office comes over to talk, or with remote work, someone rings your doorbell at home for a delivery.

Now, you can press the AirPods stem to mute yourself on the call, and when you do so, you will be notified by a friendly microphone status banner and an audio chime as well.

![muted banner][off]  

[off]: off.jpg

Then, as you would expect, you can press the AirPods stem again to unmute yourself and continue on with the meeting. And of course, you will see the banner and hear a chime on unmute.

![unmuted banner][on]  

[on]: on.jpg

Finally, you are finished with your work meeting and it's time to unwind. You pick up your iPhone, press play on your favorite Apple Music playlist, and audio switches back to your AirPods from iPhone seamlessly, so you can continue on with your day without missing a beat.

So do you want to unlock the magical AirPods experience for your application and people with AirPods be able to use your app with a breeze across platforms? In this session, we will go through how you can optimize your app to take advantage of these features and deliver exactly that. 

## AirPods automatic switching for macOS.

- Automatically switch your macOS audio to AirPods based on the audio activity  
- Uses indicators like "Now Playing" registration and input audio activity  
- Automatic participation for Apps without sandbox restrictions or those using App Sandbox  

macOS 14 supports AirPods automatic switching, between devices based on the user's intended activity. Automatic switching algorithm uses indicators like "Now Playing" registration and input audio activity to make those decisions. The good news is that all App Store apps, as well as apps outside the App Store which uses App Sandbox or opt not to have a sandbox, do not have to do anything to adopt it. They will be able to fully participate in this feature without any change needed from the developer.  
Now, let's talk through best practices for an optimal AirPods experience for people with these feature. With registering for Now Playing on macOS, the automatic routing algorithm can now know when a long-form audio is playing, for example, and allows us to make the right routing decisions.

If your app is a media or streaming app, we recommend you register for Now Playing to help us prioritize your audio accordingly.  
If you are a conference or a gaming app, we do not recommend registering for Now Playing.

We also recommend using Audio Services API to play notifications and app-specific chimes.  This would help differentiate notifications from media contents and avoid unexpected behaviors.  
If you are a conferencing app, we recommend using input mic only when conference or voice session starts and keep it open only for the duration of the live meeting or voice session.  
For media apps, we recommend you use the default route selected by the user to play audio.  
Also, avoid playing silence after the user pauses audio. If you have to play silence, we would recommend to keep it less than two seconds.

## Press to Mute and Unmute with AirPods
- (New!) Gesture on AirPods to control application's mic  
- CallKit based apps will automatically get support  
- (New!) API for communication apps not using CallKit  

With iOS 17 and macOS 14, People can now mute or unmute an application's mic during calls by simply pressing AirPods stem. Starting in iOS 17, all CallKit apps will get Press to Mute and Unmute support with absolutely no additional adoption necessary.  
For communications app not using CallKit, we have introduced new API that I will detail shortly. In all cases, iOS 17 will respond to the mute gesture by muting the uplink audio, playing a tone... And notifying the application that it has been muted. Then, when someone triggers the same gesture again, iOS 17 will unmute the uplink audio, play the tone... And notify the application that it has been unmuted. 


![unmuted - muted banner][onOff]  

[onOff]: onOff.jpg

This features adds significant convenience for people. Let me show you how quickly you can add it on iOS17.

### introducing AVAudioApplication
AVAudioApplication is a new sibling in the AVAudioSession family.  
It is a way to configure application-wide audio behaviors for your app.

Let me show you how quickly you can incorporate Press to Mute and Unmute using AVAudioApplication. First, we need to import AVFAudio. Next, get the shared instance of AVAudioApplication. Then we will need to register for mute gesture change notifications with Notification Center. This opts your application into Press to Mute and Unmute.  
These notifications will inform you when your mute state has changed by a mute gesture. From there, you introspect the AVAudioApplication input muteStateKey of the user info to determine the new state.  
After receiving this notification, you can update any internal state and UI as a result of this notification firing. Additionally, it is necessary that AVAudioApplication is updated when someone takes a mute action within your application.
As you would expect, we provide both setters and getters.
```swift
// Adopting AVAudioApplication into your App
import AVFAudio

// Get the started instance 
let instance = AVAudioApplication.shared

// Register for mute gesture notifications on Notification Center 
AVAudioApplication.inputMuteStateChangeNotification

// Key for mute state
AVAudioApplication.muteStateKey

// Updating AVAudioApplication’s mute state
instance.setInputMuted(...)

// Reading AVAudioApplication’s mute state
instance.isInputMuted
```

It is that simple to incorporate, Press to Mute and Unmute into your app on iOS 17 Moving over to the Mac.  
It is important to note that in macOS 14, Press to Mute and Unmute works a little bit differently. Just like iOS, your app will be notified when the mute state changes. However, on macOS, your application is responsible for muting any uplink audio when a gesture has been performed.

There is some additional API required to opt in, which I will detail in a moment.

Lastly, do note, for applications looking to detect muted talkers, please refer to "What's new in voice processing APIs" session for details on new API to support that.

Let's take a look at what adoption is needed on the Mac. Remember, on macOS 14, it is your application's responsibility to mute uplink audio as a result of a mute gesture.

On the shared instance of AVAudioApplication, you will need to set a Mute State Change handler. This handler will be called when someone performs a mute gesture. Here, you mute any uplink audio and update any internal state. It also gives you an opportunity to reject if the mute action is not appropriate for your application's current usage. It is recommended that this handler should not be used for UI updates, since you will continue to receive Input Mute State Change notification when mute state changes. In addition, we have provided a new property within the CoreAudio framework to help you quickly mute any input audio to your process.
```swift
// Configure the Input Mute State Change handler (macOS only)
instance.setInputMuteStateChangeHandler { isMuted in
    //...
    return didSucceed
}

// Optional: let CoreAudio mute your input for you (macOS only)
// Define the Core Audio property
var inputeMutePropertyAddress = AudioObjectPropertyAddress(
    mSelector: kAudioHardwarePropertyProcessInputMute,
    mScope: kAudioObjectPropertyScopeInput,
    mElement:kAudioObjectPropertyElementMain)

// Enable this property when you want to mute your input
UInt32 isMuted = 1; // 1 = muted, 0 = unmuted
AudioObjectSetPropertyData(kAudioObjectSystemObject,
                           &inputeMutePropertyAddress,
                           0,
                           nil,
                           UInt32(MemoryLayout.size(ofValue: isMuted),
                           &isMuted)
```

This property, when enabled, will silence any input audio to your process while continuing to perform IO as usual.

## Spatial Audio with AirPods
With the launch of iOS 16, we took this to the next level with Personalized Spatial Audio, because the way we all perceive sound is unique, based on the size and shape of our head and ears. And this push towards increased personalization has been well-received by people. With macOS 14, iOS 17 and tvOS 17, we are glad to announce continued Spatial Audio support for all three platforms.

For macOS, we support Spatial Audio for AVPlayer as well as AVSampleBufferAudioRenderer API. iOS and tvOS support Spatial Audio for AURemoteIO as well as AudioQueue API along with the above mentioned APIs.

## Spatial Audio API support

![Spatial Audio Support][SpatialAudio]  

[SpatialAudio]: spatialAudio.jpg


Do note that Spatial Audio for AudioQueue and AURemoteIO does not have an API interface. Instead, it is automatically enabled for apps that register for Now Playing.

This would present the user with an option to configure the feature via the Control Center.

## Wrap Up
- Transform your app behavior for AirPods Automatic Switching
- Press to Mute and Unmute and associated API
- Spatial Audio across platforms

# Check out also 
[Immerse your app in Spatial Audio - WWDC21](https://developer.apple.com/videos/play/wwdc2021/10265/)  
[Reaching the Big Screen with AirPlay 2. - WWDC19](https://developer.apple.com/videos/play/wwdc19/501/)  
[Becoming a now playable app (Sample Code) -  WWDC19](https://developer.apple.com/documentation/mediaplayer/becoming_a_now_playable_app)  

