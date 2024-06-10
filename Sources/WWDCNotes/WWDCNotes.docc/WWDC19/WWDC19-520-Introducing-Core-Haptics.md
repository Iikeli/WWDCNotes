# Introducing Core Haptics

Core Haptics lets you design fully customized haptic patterns with synchronized audio. See examples of how haptics and audio enables you to create a greater sense of immersion in your app or game. Learn how to create, play back, and share content, and where Core Haptics fits in with other audio and vibration APIs.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/520", purpose: link, label: "Watch Video (29 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

[`CoreHaptics`][CHDocs] is a new framework in iOS 13.

It’s an event based audio and haptic rendering API for iPhone (in short, a synthesizer).

Compatible with iPhone 8 or later.

## `CoreHaptics` vs `UIFeebackGenerator`

`CoreHaptics` is not a replacement of [`UIFeebackGenerator`][FGDocs], Apple suggests to keep using it for `UIKit` controls, reactions, and similar.

![][chVSfgImage]

## Sound

We can have a tight connection between audio and haptic. 

This has been heavily used internally at Apple for the following applications:

![][devicesImage]

(Yes, all of them have sound!).

## Game Examples

This framework is very important for games:

- for boosting feeling
- simulate physical contact (think of a tennis game, every time the racket hits the ball)

## Classes

Two kinds:

![][classesImage]

- Content classes represent the content itself.
- Playback classes play the content.

### [`CHHapticEvent`][CHHEDocs]

The basic, indivisible content element.

Each element has a time (period), a type, and, optionally, some parameters.

Multiple Haptic events can overlap, the outcome will be a mix of all of them.

### Haptic types

- **Haptic Transient**: momentary and instantaneous
- **Haptic Continuos/Audio Continuos**: lasts longer than transient type, with more parameters to toggle
- **AudioCustom**: we provide the audio to be played in sync with other haptic types

All the `CHHApticEvents`, are then grouped into **Haptic Patterns**, [`CHHapticPattern`][CHHPDocs].

### [`AHAP`][AHAPDocs] (Apple Haptic Audio Pattern)

- New file format which describes a (Haptic) pattern as text.
- It uses JSON.
- Can be used with Swift Codable.

[CHDocs]: https://developer.apple.com/documentation/corehaptics
[FGDocs]: https://developer.apple.com/documentation/uikit/uifeedbackgenerator
[CHHEDocs]: https://developer.apple.com/documentation/corehaptics/chhapticevent
[CHHPDocs]: https://developer.apple.com/documentation/corehaptics/chhapticpattern
[AHAPDocs]: https://developer.apple.com/documentation/corehaptics/representing_haptic_patterns_in_ahap_files

[chVSfgImage]: chVSfg.png
[devicesImage]: devices.png
[classesImage]: classes.png