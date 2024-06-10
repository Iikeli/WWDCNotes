# Swan's Quest, Chapter 3: The notable scroll

Swift Playgrounds presents "Swan’s Quest,” an interactive adventure in four chapters for all ages. Calling all musicians! In this chapter, our Hero has found a mysterious scroll of music, and only you can help decode it. (Don’t worry if you can’t read music, our clever Lizard is standing by to assist. It’s sure to be a note-worthy experience.)

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10683", purpose: link, label: "Watch Video (5 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> Swan's Quest is a series of challenges where each chapter has a specific programming challenge for you that will build on the prior chapters.

Download the `.playgroundbook` [here][swdwl]. 

## Melody

In this challenge we will play a melody with different length of tones, and we will need to implement this ourselves.

## Solution

### Advice from the Lizard

- main

```swift
let notes: [Note] = [.quarter(.a4), .quarter(.b4), .quarter(.c4)]
```

- music

```swift
public protocol PitchProtocol {
    
    /// The sonic frequency of this Pitch
    var frequency: Double { get }
    
}

public protocol NoteProtocol {
    associatedtype PitchType: PitchProtocol
    
    /// Play this Note through a ToneOutput
    var tone: Tone { get }
    
    /// The duration of this Note as a multiple of quarter notes, e.g., a half note would equal 2.0, an eight note
    var length: Float { get }
    
    /// Length of the smallest Note supported
    static var shortestSupportedNoteLength: Float { get }
    
    /// Subdivide into a series pitches, according to the shortest supported note
    func subdivide() -> [PitchType]
}

public enum Note : NoteProtocol {
    case quarter(Pitch)
    case half(Pitch)
    
    /// Play this Note through a ToneOutput
    public var tone: Tone {
        switch self {
        case .quarter(let pitch), .half(let pitch):
            return Tone(pitch: pitch.frequency, volume: 0.3)
        }
    }
    
    /// The duration of this Note as a multiple of quarter notes, e.g., a half note would equal 2.0, an eight note
    public var length: Float {
        switch self {
        case .quarter:
            return 1.0
        case .half:
            return 2.0
        }
    }
    
    /// Length of the smallest Note supported
    public static var shortestSupportedNoteLength: Float {
        return Note.quarter(.a4).length
    }
    
    /// Subdivide into a series pitches, according to the shortest supported note
    public func subdivide() -> [Pitch] {
        switch self {
        case .quarter(let pitch):
            return [pitch]
        case .half(let pitch):
            return [pitch, pitch]
        }
    }
}

public enum Pitch : Double, PitchProtocol {
    case c3 = 261.63
    case d3 = 293.66
    case e3 = 329.63
    case f3 = 349.23
    case g3 = 392.00
    case a4 = 440.0
    case b4 = 493.88
    case c4 = 523.25
    
    /// The sonic frequency of this Pitch
    public var frequency: Double {
        return self.rawValue
    }
}

```

### Perdomance at Swan Hall

```swift
func performance(owner: Assessable) {
    let toneOutput = ToneOutput()
    let quarterPitches: [Pitch] = [.e3, .e3, .f3, .g3, .g3, .f3, .e3, .d3, .c3, .c3, .d3, .e3, .e3, .d3]
    let halfNotePitch: Pitch = .d3
    let notes: [Note] = quarterPitches.map{ Note.quarter($0) } + [Note.half(halfNotePitch)]
    
    var pitches: [Pitch] = notes
        .flatMap { $0.subdivide() }
        .reversed()
    
    let interval = TimeInterval(Note.shortestSupportedNoteLength * 0.5)
    Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { timer in
        guard let pitch = pitches.popLast() else {
            toneOutput.stopTones()
            timer.invalidate()
            owner.endPerformance()
            return
        }
        
        toneOutput.play(tone: Tone(pitch: pitch.frequency, volume: 0.3))
    }
    owner.endPerformance()
}
```

[swdwl]: https://developer.apple.com/sample-code/swift/swans-quest/the-notable-scroll.zip