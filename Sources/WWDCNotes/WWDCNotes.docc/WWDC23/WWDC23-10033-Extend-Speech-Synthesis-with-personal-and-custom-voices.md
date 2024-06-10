# Extend Speech Synthesis with personal and custom voices

Bring the latest advancements in Speech Synthesis to your apps. Learn how you can integrate your custom speech synthesizer and voices into iOS and macOS. We’ll show you how SSML is used to generate expressive speech synthesis, and explore how Personal Voice can enable your augmentative and assistive communication app to speak on a person’s behalf in an authentic way.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10033", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(MortenGregersen)
   }
}



Speaker: Grant Maloney, Accessibility Engineer

## Explore Speech Synthesis Markup Language (SSML)

What is SSML?
- A W3C standard for speech
- Declarative XML document
- Supported on all Platforms

```swift
let ssml = """
    <speak>
        Hello
        <break time="1s" />
        <prosody rate="200%">nice to meet you!</prosody>
    </speak>
"""

guard let ssmlUtterance = AVSpeechUtterance(ssmlRepresentation: ssml) else {
    return
}

self.synthesizer.speak(ssmlUtterance)
```

## Implement a synthesis provider

### What is a speech synthesizer and how does it work?

> A speech synthesizer receives some text and information about desired speech properties in the form of SSML and provides an audio representation of that text.

> Speech Synthesis provider audio unit extensions will be embedded in a host app and will receive speech requests in the form of SSML. The extension will be responsible for rendering audio for the SSML input and optionally returning markers indicating where words occur within those audio buffers. The system will then manage all playback for that speech request. You don't need to handle any audio session management; it's managed internally by the Speech Synthesis Provider framework.

### Creating a new voice

Create a new Audio Unit Extension app project in Xcode, then select the "Speech Synthesizer" Audio Unit Type and provide a four character subtype identifier for your synthesizer, as well as a four character identifier for you as a manufacturer.

Use an App Group to communicate between the extension and the host app.

```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var speechVoices: [AVSpeechSynthesisProviderVoice] {
        get {
            let voices: [String : String] = groupDefaults.value(forKey: "voices") as? [String : String] ?? [:]
            return voices.map { key, value in
                return AVSpeechSynthesisProviderVoice(name: value,
                                                identifier: key,
                                          primaryLanguages: ["en-US"],
                                        supportedLanguages: ["en-US"] )
            }
        }
    }
}
```

#### The synthesis provider

When some text should be synthesized this function will be called with the SSML.

```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override func synthesizeSpeechRequest(speechRequest: AVSpeechSynthesisProviderRequest) {
        currentBuffer = getAudioBuffer(for: speechRequest.voice, with: speechRequest.ssmlRepresentation)
        framePosition = 0
    }

    public override func cancelSpeechRequest() {
        currentBuffer = nil
    }
}
```

### The render block

```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var internalRenderBlock: AUInternalRenderBlock {
       return { [weak self]
           actionFlags, timestamp, frameCount, outputBusNumber, outputAudioBufferList, _, _ in
           guard let self else { return kAudio_ParamError }

           // This is the audio buffer we are going to fill up
           var unsafeBuffer = UnsafeMutableAudioBufferListPointer(outputAudioBufferList)[0]
           let frames = unsafeBuffer.mData!.assumingMemoryBound(to: Float32.self)
                
           var sourceBuffer = UnsafeMutableAudioBufferListPointer(self.currentBuffer!.mutableAudioBufferList)[0]
           let sourceFrames = sourceBuffer.mData!.assumingMemoryBound(to: Float32.self)

           for frame in 0..<frameCount {
               if frames.count > frame && sourceFrames.count > self.framePosition {
                   frames[Int(frame)] = sourceFrames[Int(self.framePosition)]
                   self.framePosition += 1
                   if self.framePosition >= self.currentBuffer!.frameLength {
                       break
                   }
               }
           }
                
           return noErr
       }
    }
}
```

### Sample app for purchasing voices

Take note of the `AVSpeechSynthesisProviderVoice` function `updateSpeechVoices()`. That is how your app can signal that the set of available voices for your synthesizer has changed and the system voice list should be rebuilt.

```swift
struct ContentView: View {
    
    @State var purchasedVoices: [WWDCVoice] = []
    
    var body: some View {
        List {
            MyAwesomeVoicesSection
            PurchasedVoicesSection
        }
    }
    
    func purchase(voice: WWDCVoice) {
        // Append voice to list of purchased voices
        purchasedVoices.append(voice)
        
        // Inform system of change in voices
        AVSpeechSynthesisProviderVoice.updateSpeechVoices()
    }
}
```

### Listen for changes in available system voices

```swift
struct ContentView: View {
    @State var systemVoices: [AVSpeechSynthesisVoice] = AVSpeechSynthesisVoice.speechVoices()
    
    var body: some View {
        List {
            MyAwesomeVoicesSection
            PurchasedVoicesSection
            Section("System Voices") {
                ForEach(systemVoices.filter { $0.language == "en-US" }) { voice in
                    Text(voice.name)
                }
            }
        }
        .onReceive(NotificationCenter.default
            .publisher(for: AVSpeechSynthesizer.availableVoicesDidChangeNotification)) { _ in
                systemVoices = AVSpeechSynthesisVoice.speechVoices()
        }
    }
}
```

## Use Personal Voice

> People can now record and recreate their voice on iOS and macOS using the power of their device.

> Your Personal Voice is generated on the device and not on a server. This voice will appear amongst the rest of the System voices and can be used with a new feature called Live Speech. Live Speech is a type-to-speak feature on iOS, iPadOS, macOS, and watchOS that lets a person synthesize speech with their own voice on the fly.

> You can request access to synthesize speech with these voices using a new request authorization API for Personal Voice. Keep in mind that usage of Personal Voice is sensitive and should be primarily used for augmentative or alternative communication apps.

```swift
struct ContentView: View {
    @State private var personalVoices: [AVSpeechSynthesisVoice] = []

    func fetchPersonalVoices() async {
        AVSpeechSynthesizer.requestPersonalVoiceAuthorization() { status in
            if status == .authorized {
                personalVoices = AVSpeechSynthesisVoice.speechVoices().filter { $0.voiceTraits.contains(.isPersonalVoice) }
            }
        }
    }
}
```

When authorized you can use the Personal Voice:

```swift
func speakUtterance(string: String) {
    let utterance = AVSpeechUtterance(string: string)
    if let voice = personalVoices.first {
        utterance.voice = voice
        syntheizer.speak(utterance)
    }
}
```
