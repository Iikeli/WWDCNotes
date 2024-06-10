# Swan's Quest, Chapter 4: The sequence completes

Swift Playgrounds presents "Swan’s Quest,” an interactive adventure in four chapters for all ages. It’s time for the grand finale: You’ve honed your skills with tones, but in this chapter our Hero needs to sequence multi-part harmony.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10684", purpose: link, label: "Watch Video (8 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> Swan's Quest is a series of challenges where each chapter has a specific programming challenge for you that will build on the prior chapters.

Download the `.playgroundbook` [here][swdwl]. 

## Sample Instruments

In this challenge we will need to play multiple tones at the same time, we will need a step sequencer in order to do that.

### What is a step sequencer?

TL;DR: In short it lets us play multiple tones at the same time using the same timer.

A sequencer is a multi-track timing loop: i
- it's divided into equal chunks/steps which play in sequence over a predefined duration. 
- each track represents an instance of a pitched instrument.

Sequencers are also used with non pitched instruments like percussion.

## Solution

### Advice from the Lizard

- main

```swift
func performance(owner: Assessable) {
    playInstrument(.piano, note: MIDINotes.c2)
    owner.endPerformance()
}

```

- sequencer

```swift
public protocol MIDINoteProtocol {
    
    /// note as an 8-bit MIDI code
    var midiCode: UInt8 { get }
}

public protocol TrackProtocol {
    associatedtype NoteType : MIDINoteProtocol
    
    /// The kind of instrument that the track sequences
    var instrument: Instrument.Kind { get }
    
    /// Number of beats contained in the sequence
    var length: Int { get }
    
    /// MIDI code for the sequence frame
    func note(for frame: Int) -> NoteType
}

// The Wizard has provided a MIDI Notes implementation for you.
public enum MIDINotes : UInt8, MIDINoteProtocol {
    case rest = 0
    
    case c1  = 24
    case cs1 = 25
    case d1  = 26
    case ds1 = 27
    case e1  = 28
    case f1  = 29
    case fs1 = 30
    case g1  = 31
    case gs1 = 32
    case a1  = 33
    case as1 = 34
    case b1  = 35
    
    case c2  = 36
    case cs2 = 37
    case d2  = 38
    case ds2 = 39
    case e2  = 40
    case f2  = 41
    case fs2 = 42
    case g2  = 43
    case gs2 = 44
    case a2  = 45
    case as2 = 46
    case b2  = 47
    
    case c3  = 48
    case cs3 = 49
    case d3  = 50
    case ds3 = 51
    case e3  = 52
    case f3  = 53
    case fs3 = 54
    case g3  = 55
    case gs3 = 56
    case a3  = 57
    case as3 = 58
    case b3  = 59
    
    case c4  = 60
    case cs4 = 61
    case d4  = 62
    case ds4 = 63
    case e4  = 64
    case f4  = 65
    case fs4 = 66
    case g4  = 67
    case gs4 = 68
    case a4  = 69
    case as4 = 70
    case b4  = 71
    
    case c5  = 72
    case cs5 = 73
    case d5  = 74
    case ds5 = 75
    case e5  = 76
    case f5  = 77
    case fs5 = 78
    case g5  = 79
    case gs5 = 80
    case a5  = 81
    case as5 = 82
    case b5  = 83
    
    /// note as an 8-bit MIDI code
    public var midiCode: UInt8 {
        return self.rawValue
    }
}

public class Track : TrackProtocol {
    public var instrument: Instrument.Kind
    public var length: Int
    public var notes: [MIDINotes]
    
    public init(_ instrument: Instrument.Kind, length: Int = 0, notes: [MIDINotes]) {
        self.instrument = instrument
        self.length = length
        self.notes = notes
    }
    
    public func note(for frame: Int) -> MIDINotes {
        guard frame < notes.count else {
            return .rest
        }
        return notes[frame]
    }
}
```

### Perfomance at Swan Hall

```swift
let bass: [MIDINotes] = [
    .g3, .rest, .g2, .rest,
    .c3, .g3, .c4, .rest,
    .b4, .rest, .as4, .rest,
    .f2, .rest, .f3, .e3,
    .d3, .rest, .g2, .rest,
    .c4, .rest, .rest, .rest,
    .a3, .rest, .fs3, .rest,
    .f3, .d3, .c4, .a3
]

let treble: [MIDINotes] = [
    .rest, .b3, .c4, .d4,
    .e4, .rest, .rest, .d4,
    .e4, .a4, .g4, .e4,
    .d4, .c4, .a3, .rest,
    .rest, .c4, .e4, .f4,
    .g4, .rest, .rest, .a4,
    .g4, .e4, .c4, .e4,
    .d4, .rest, .rest, .rest
]

func performance(owner: Assessable) {
    let numberOfBeats = 32   // two bars of 4/4
    let duration = 16.0      // seconds
    
    let tracks = [
        Track(.bassGuitar, length: numberOfBeats, notes: bass), 
        Track(.piano, length: numberOfBeats, notes: treble)
    ]
    
    let interval = duration / Double(numberOfBeats)
    var index = 0
    Timer.scheduledTimer(withTimeInterval: interval, repeats: true, block: { timer in
        for track in tracks {
            playInstrument(track.instrument, note: track.note(for: index))
        }
        
        index += 1
        if index >= numberOfBeats {
            timer.invalidate()
            owner.endPerformance()
        }
    })
}
```

[swdwl]: https://developer.apple.com/sample-code/swift/swans-quest/the-sequence-completes.zip