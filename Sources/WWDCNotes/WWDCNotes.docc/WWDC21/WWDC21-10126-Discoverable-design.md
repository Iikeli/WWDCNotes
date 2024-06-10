# Discoverable design

Discover how you can create interactive, memorable experiences to onboard people into your app. We’ll take you through discoverable design practices and learn how you can craft explorable, fun interfaces that help people grasp the possibilities of your app at a glance. We’ll also show you how to apply this methodology to personalize your content and make your app easy to customize.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10126", purpose: link, label: "Watch Video (32 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



- You need an interface that stands on its own, without heavy-handed (onboarding) tutorials
- learning by doing is a lot more fun and effective than reading a list of instructions (people skip or don't remember onboarding tutorials)

Five fundamentals to make your app more discoverable:

## 1. Prioritize important features (so you can make the most important ones visible)

- essential parts of the app should be immediately visible to people
- for non-essential parts, it's ok to require navigation to reach
- make it easy to reach the parts of the app people use frequently

## 2. Provide visual cues (use words and symbols people are familiar with)

- do not use hamburger menus, people don't know what's inside (use tab bars or others)
- minimal ≠ usable and simple, too minimal means we increase the risk that people won't find features
- know your audience, will help you understand what needs to be spelled out and what can be inferred by people
- think about who is your product for
- guide with words and visuals

## 3. Hint at gestures

- making use of gestures in your app will make your app feel more fluid and responsive than navigating through with only discrete taps
- use the accepted gesture for the platform
- if you're inventing a new gesture, try to mimic real-life interactions
- use gestures as a shortcut, not a replacement
- e.g. hint to a swipe down dismissal when they user taps the back/close button by performing a sliding down animation

## 4. Organize by behavior (group content to fit people's behavior)

- use personalization
- organize content into a dedicated section that will surface suggestions from a machine-learning-powered recommendation engine
- visualize organization (e.g. by making the categories clear at a glance)

## 5. Convey a sense of control over personalized content

- let people provide explicit feedback (about the content)
- disclose implicit feedback (a.k.a. feedback we gather without people consciously asking for it, e.g. if we suggest content based on a previously purchased item)
- give control over recommendations
