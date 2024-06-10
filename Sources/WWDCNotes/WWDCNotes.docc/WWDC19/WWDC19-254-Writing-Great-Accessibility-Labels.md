# Writing Great Accessibility Labels

Great accessibility labels are the difference between someone using and loving your app or someone deleting your app. Experience VoiceOver as demonstrated by an Apple Accessibility engineer as she navigates complex UI and demonstrates how descriptive labels are an easy way to ensure your app is for everyone.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/254", purpose: link, label: "Watch Video (10 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



It’s all about the context.

Let’s take the “add button” for example: it’s very important to know that this button does. “adds a note”, or “adds an item to cart”, and more. 

Disclose exactly which item as well.

## Best practices

- Name all the important ui elements.
- Don’t include the element type: things like button “add button“, as Siri will read that element as “add button, button.” (because Siri always says what every element is).
- In case of a toggle element, remember to update its accessibility label based on the current state (like turn on light, turn off light).
- If the same element is present multiple times, add more context on each element (e.g. add chocolate to cart, add napkin to cart, add banana to cart, “add to cart“ is too generic)
- Avoid redundancy “play song” “skip song” “next song”, if the context is clear, drop “song”
- Be succinct, no verbose. In proper cases it’s ok to be verbose
