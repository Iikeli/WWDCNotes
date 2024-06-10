# Swan's Quest, Chapter 2: A time for tones

Swift Playgrounds presents "Swan’s Quest,” an interactive adventure in four chapters for all ages. In this chapter, our Hero needs your help decoding the Swan’s scroll. Call forth the best of your audio abilities on this one — you’re going to need them.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10682", purpose: link, label: "Watch Video (5 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> Swan's Quest is a series of challenges where each chapter has a specific programming challenge for you that will build on the prior chapters.

Download the `.playgroundbook` [here][swdwl]. 

## Tones

For this first challenge, a `SPCAudio` `PlaygroundModule` has been built, this module is part of the [Create Quest][createQuestDwl] Playground Book.

In this challenge you will need to play a tone with the secret code given in the previous quest. All of this is done via a `Timer` and a `SPCAudio`'s' `Tone`.

## Solution

### Advice from the Lizard

```swift
let toneOutput = ToneOutput()

func performance(owner: Assessable) {
    
    let tones = [
        Tone(pitch: 440.00, volume: 0.3),
        Tone(pitch: 493.88, volume: 0.3),
        Tone(pitch: 523.25, volume: 0.3) 
    ]
    
    var toneIndex = 0
    Timer.scheduledTimer(withTimeInterval: 0.4, repeats: true) { timer in
        guard toneIndex < tones.count else {
            toneOutput.stopTones()
            timer.invalidate()
            owner.endPerformance()
            return
        }
        
        toneOutput.play(tone: tones[toneIndex])
        toneIndex += 1
    }
    // Let it be known your performance has ended.
    owner.endPerformance()
}
```

### Performance at Swan Hall

```swift
func performance(owner: Assessable) {
    let toneOutput = ToneOutput()
    
    // TODO: Code your performance here.
    let tones = [261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88, 523.25].map({ Tone(pitch: $0, volume: 0.3)})
    
    var toneIndex = 0
    Timer.scheduledTimer(withTimeInterval: 0.4, repeats: true) { timer in
        guard toneIndex < tones.count else {
            toneOutput.stopTones()
            timer.invalidate()
            owner.endPerformance()
            return
        }
        
        toneOutput.play(tone: tones[toneIndex])
        toneIndex += 1
    }
    
    // Let it be known your performance has ended.
    owner.endPerformance()
}
```

[swdwl]: https://developer.apple.com/sample-code/swift/swans-quest/a-time-for-tones.zip
[createQuestDwl]: https://developer.apple.com/sample-code/swift/swans-quest/quest-create.zip