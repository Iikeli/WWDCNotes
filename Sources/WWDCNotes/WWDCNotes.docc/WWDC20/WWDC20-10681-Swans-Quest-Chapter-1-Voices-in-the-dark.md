# Swan's Quest, Chapter 1: Voices in the dark

Swift Playgrounds presents "Swan’s Quest,” an interactive adventure in four chapters for all ages. In this chapter, our Hero must navigate a dark cave — and the only way to light the torches is to make them accessible.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10681", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> Swan's Quest is a series of challenges where each chapter has a specific programming challenge for you that will build on the prior chapters.

Download the `.playgroundbook` [here][swdwl]. 

## VoiceOver

To complete the first challenge, you're going to use VoiceOver.

### What is VoiceOver?

- VoiceOver is Apple's screen reader
- VoiceOver provides information about text, controls, and other elements as you move through an app's interface using text-to-speech, Braille, or a combination of the two.

In this chapter, you're gonna write code using VoiceOver.

## Getting Started

### Toggling VoiceOver

#### iOS/iPadOS

To make it easy for us to toggle VoiceOver: go to `Settings > Accessibility > Accessibility Shortcut` and choose VoiceOver.

This will enable you to toggle VoiceOver by pressing three times the home button or the side button (depending on your iPhone form factor).

> Alternatively, you can also ask Siri to turn on/off VoiceOver

#### macOS

Use `⌘ + F5`, or triple-click the TouchID button to activate VoiceOver.

- You can move the VoiceOver cursor through the screen by pressing `⌃ + ⌥` plus either the Left or Right Arrow keys.
- You can navigate up and down by using `⌃ + ⌥` plus the Up or Down Arrow keys. 
- To enter a group of controls or elements such as the Source Code Editor in Playgrounds, press `⌃ + ⌥ + ⇧` plus the Down Arrow to do what it's called _interacting_. 
- To stop interacting, to get out of a group of controls, press `⌃ + ⌥ + ⇧` plus the Up Arrow key. 
- Use `⌃ + ⌥` plus the Space Bar to activate a control.

## Getting Familiar with Voice Over

Once activated, a black rectangle around the selected control will appear, this is called the VoiceOver cursor, and it confirms visually which item has VoiceOver's focus. 

You can flick left or right with one finger to move the cursor over different elements on the screen.

You can drag your finger around the screen to move the cursor over elements more quickly.

To leave an app either:
- press the Home button 
- slide one finger up from the bottom edge of the screen until you hear the first sound. If you pause there, VoiceOver will say, "Lift for Home.". Lift for Home.

## SPCAccessibility

For this first challenge, a `SPCAccessibility` `PlaygroundModule` has been built, this module is part of the [Create Quest][createQuestDwl] Playground Book.

`SPCAccessibility` has four on-screen elements that form its foundation: 

- `Graphic`
- `Button`
- `Sprite`
- `Label` 

All of them inherit from `BaseGraphic`, which contains properties for accessibility support.

An example of accessibility property is `AccessibilityHints` property: setting this property indicates that this screen element should be read by VoiceOver, and makes it visible to other assistive services.

To support VoiceOver, these are the values you need to supply: 
- `makeAccessibilityElement` tells VoiceOver it should stop the cursor on this item. 
- `accessibilityLabel` is what VoiceOver reads as it stops on the item.

## To pass the first challenge..

..you will need to make the screen elements accessible.

## Solution

```swift
cave.torch1.accessibilityHints?.accessibilityLabel = "First torch"
cave.torch2.accessibilityHints?.accessibilityLabel = "Second torch"
cave.torch3.accessibilityHints?.accessibilityLabel = "Third torch"
cave.torch4.accessibilityHints?.accessibilityLabel = "Fourth torch"
cave.torch5.accessibilityHints?.accessibilityLabel = "Fifth torch"
cave.torch6.accessibilityHints?.accessibilityLabel = "Sixth torch"
```

[swdwl]: https://developer.apple.com/sample-code/swift/swans-quest/voices-in-the-dark.zip
[createQuestDwl]: https://developer.apple.com/sample-code/swift/swans-quest/quest-create.zip