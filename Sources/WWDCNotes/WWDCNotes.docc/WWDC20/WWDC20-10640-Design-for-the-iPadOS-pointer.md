# Design for the iPadOS pointer

Bring the power of the pointer to your iPad app: We’ll show you how Apple's design team approached designing the iPadOS pointer to complement touch input, and how you can customize and refine pointer interactions in your app to make workflows more efficient and gratifying. Discover how the pointer’s adaptive precision enables people to quickly and confidently target interface elements regardless of their size. We’ll also share some best practices on adapting the pointer to complement your app's unique needs including how to select pointer effects and design pointer shapes, integrate trackpad gestures, and keyboard modifiers.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10640", purpose: link, label: "Watch Video (41 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Adaptive Precision

While on the mac the pointer gives you increased precision compared to touch. On iPad it's helpful to actually reduce the precision of the pointer to match the user interface.  

This concept of dynamically adjusting the precision of the pointer to match the precision of the interface is called, **Adaptive Precision**.

A traditional pointer is drawn _over_ the entire interface often obscuring the control you're interacting with, the iPad pointer lets you have the controls you're interacting with completely unobstructed: when the pointer snaps to a control, the pointer shifts from being in front of the app to being behind the buttons icon or label. 

## Two pointers

![][pointersImage]

When you move the pointer around on screen, you're actually moving two pointers: 

- the one you see on screen
- an invisible one, that tracks the true position of the pointer.

The latter is called **Model pointer**: it is used to decide which item the pointer is hovering over and it takes advantage of the generous padding that buttons have on iPadOS to make them easier to tap with the finger. 

Once you lift your finger the pointer centers itself on the current active control, this is called **Recentering**.

When swiping, the iPad pointer can figure out which control you were aiming for and move there automatically: this is called **Magnetism**. Magnetism scans the interface to find the control you most likely want to target. 

## Pointer Effects

When the iPadOS pointer hovers over an interactive element, both the appearance and the behavior of the pointer and the interactive element become one, bringing focus to the item that is being targeted, this is called **Pointer Effect**.

iPadOS provides three pointer effects, based on context and content type, and you can create your own as well.

### Highlight Effect

![][highlightImage]

- Used for small controls that don't have a background
- Default effect for bar buttons and tab bars

### Lift Effect

![][liftImage]

- Used for medium sized elements that already have a background
- Default effects for app icons and control center modules

### Hover Effect

![][hoverImage]

- Used for larger objects that would behave poorly if the pointer were to morph into their shape
- Customizable: it can be just a scale of the object and a shadow to lift it, a color tint added to the object, or any combination of the three

### Tips

- When to use which effect: start by selecting the [`.automatic`][autoDoc] effect: the system uses a combination of rules like the object type and location, and the object size and shape to decide the best effect for it.
- Make group of elements pointer effects consistent
- Use an height of 37 points for toolbar items
- Avoid the highlight effect around rectangular objects
- define a good hit region for each element: not too small nor too big. A general rules is having a padding of about 12 points around elements that include a bezel, 24 points around elements without a bazel:  
![][paddingImage]

- don't leave hit regions gaps between elements in the same group
- make sure elements that use the lift effect don't have the shadow clipped
- when using the Lift effect, provide the correct size and corner radius
- when using the Hover effect, you can customize the effect scale, shadow, and/or tint color

## Designing custom pointers

- make sure your shapes are simple and easy to understand: the shape of the pointer informs people the action they can take in the current context
- use solid shapes as much as possible
- if you can't use solid shapes, use heavy strokes (4.5 points)
- make your shape visual weight similar to the default pointer, which is 19 points in diameter.

[autoDoc]: https://developer.apple.com/documentation/uikit/uipointereffect/automatic

[pointersImage]: WWDC20-10640-pointers
[highlightImage]: WWDC20-10640-highlight
[liftImage]: WWDC20-10640-lift
[hoverImage]: WWDC20-10640-hover
[paddingImage]: WWDC20-10640-padding
