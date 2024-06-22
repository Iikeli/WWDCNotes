# Introducing Expanded Subscriptions in iTunes Connect

See what's new in subscriptions. Learn how our improvements give you more flexibility and control over pricing, and provide powerful incentives to engage and retain your customers.

@Metadata {
   @TitleHeading("WWDC16")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc16/301", purpose: link, label: "Watch Video (34 min)")

   @Contributors {
      @GitHubUser(antonio081014)
   }
}



## What is an In-App Purchase

Digital content or service bought within app:

- Consumable
- Non-consumable
- Non-renewable subscriptions
- Auto-renewable subscriptions

## Auto-Renewable Subscriptions

### Increased proceeds

Proceeds goes from 70% to 85% for subscriptions over a year.

### Subscription Groups

Only one subscription in a single group can be selected, if the app needs to support multiple active subscriptions at the same time, then multiple subscription groups should be created.

In the same group, the level of subscription could be offered for the different subscriptions, thus, user could upgrade/downgrade subscription level.

## Territory Pricing

Price can be customized for each territory.

### Customer Retention

Using _Push Notification_ and highly customized _Email_ would be a good idea to keep attracting existing customers.

#### Preserve Price

Multiple ways to manipulate the prices:

- Initial price could be offered for the soft launch (Early adopters).
- A different price could be offered after trial period.
- Increase the price without changing existing customers, after a certain date.
- Increase the price without changing early adopters price, after a certain date.
- Early adopters price could also be changed. (That will be effective immediately.)