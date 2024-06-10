# Principles of spatial design

Discover the fundamentals of spatial design. Learn how to design with depth, scale, windows, and immersion, and apply best practices for creating comfortable, human-centered experiences that transform reality. Find out how you can use these spatial design principles to extend your existing app or bring a new idea to life.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10072", purpose: link, label: "Watch Video (21 min)")

   @Contributors {
      @GitHubUser(mbbischoff)
   }
}



- speakers:
	- [[Nathan Gitter]]
	- [[Amy DeDonato]]
Goal: Learn to design for a spatial operating system.

### Familiar
- Common elements like sidebars, tabs, and search fields.
- We place interfaces within windows so people can see them and use them.
- Windows on [[visionOS]]
	- Windows
		- Glass material provides context, awareness of surroundings, and adapts to lighting conditions
		- Controls to move, close, and resize windows
	- Sizing
		- Designed to be comfortable for the content
			- Safari is tall by default
			- Keynote is wide
		- Resizable
		- Tab bars and toolbars push outside the window and are layered above it
		- Or use multiple sections to set content away from controls
			- See Safari’s implementation of the toolbar and sidebar
		- Apps can have multiple windows
			- In Keynote, the slide and presenter d display are both visible separately.
		- Ideally keep your interface in a single window if you can because multiple can become a lot for users to manage.
	- [[Points]]
		- Points are the way we specify the size of interface elements so they can adapt tom future scenes
		- **As windows get further away, they get larger to remain legible.**
		- Design your interface with points and it will work at any distance.

### Human-centered
- Good design is always human-centered.
- Field of view
	- It’s easiest to see things in the center.
	- The field of view is wide, so prefer landscape layouts
	- Example:
		- Safari’s tabs view expands in landscape and tilts tabs toward the user.
- Ergonomics
	- Place objects comfortably in all dimensions
	- When placeing your own content, place it relative to the user’s head and the way they’re facing 
		- Think about the laying back on the couch use case
	- Place content 
		- A bit further than arm’s reach to discourage direct interaction
		- Not too high or too low
		- Not everyone will be able to move around
		- Avoid anchoring content to people’s view, instead anchor it to their space
- Movement
	- Require minimal movement
	- People should be able to use your app without moving
	- When they do move, they can press and hold the [[Digital Crown]] to recenter
		- Your app doesn’t need to have a way to recenter or reset the scene.

### Dimensional
- Design your app to work in any amount of space
- Don’t constrain your app by the physical space available.
- With windows, you don’t need tow worry about how.
- Dimming allows windows to dim the background and focus on content.
- If you do need more space for immersion, you can create a [[Full Space]].
- Create hierarchy with depth.
- Example: TV app
	- Playback controls are small and nearby.
	- Movie is large and far away.
- Use light an shadow to reinforce depth.
- Any custom objects in your app should cast shadows.
- Prefer subtle depth.
	- Modals push back windows very slighty.
- Not everything needs depth.
	- Keep text flat when using it as an interface element.
- Explore different scaels for your content
	- Most items should probably be at their natural scale.
### Immersive
- Immersion spectrum
	- [[Shared Space]]
		- Try to start your app here.
	- [[Full Space]]
		- Keynote uses this when you’re rehearsing the presentation to show you a full theater.
- Essential tips
	- Guide people’s focus toward parts of your experience that matter most.
		- In [[Mindfulness App]], the flower is the focus point, until the petals expand outward when it’s time for deeper focus.
	- Blend thoughtfully with reality
		- Use the shape of the room to blend content meaningfully with it.
		- When blending entire scens into someone’s space, use smooth edges.
	- Make things feel alive.
		- Subtle animation like water rippling on a lake or clouds in the sky.
	- Create atmosphere with sound.
		- see: [[Explore immersive sound design]]
	- Do more with less
		- In [[Cinema]], a subtle light on the floor and ceiling gives the user of being in one
- Comfort
	- Fade out content in motion
	- Provide a clear way in and out of your immersive experience.
		- Expand and collapse arrows.
### Authentic
- Apps on this platform are better when they can hang around longer.
- Think about how you can make your app worthwhile, engaging, and distinct enough t
- Find a key moment
	- Example: Photos, the key moment is selecting a panorama that is fully immersive.
	- Enhance a moment with depth and scale.
	- Transport someone
	- Evoke a feeling.
