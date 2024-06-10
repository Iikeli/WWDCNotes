# Explore Safari Web Extension improvements

Learn how you can extend Safari’s functionality with Safari Web Extensions. We’ll introduce you to the latest WebExtension APIs, explore non-persistent background page support — a particularly relevant topic if you’re developing for iOS — and discover how you can use the Declarative Net Request WebExtensions API to block content on the web. Finally, we’ll show you how to customize tabs in Safari 15.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10027", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Three new extensions API:

- non-persistent background pages - new way to structure your extension for better performance.
- declarative content blocking 
- customizing new tabs

## Non-persistent background pages

- introduction:
  - Some extensions have a script that run in the background of the browser called a background page
  - a background page doesn't have any visible UI, but it can react to events like a tab opening or a message from another part of the extension
  - a persistent background page never closes
  - persistent background pages are like these invisible tabs that a user can never close, and they eat up memory and increase CPU usage

- New this year: non-persistent background
  - mandatory in iOS (because of resource constraints)
  - event driven (the page registers event listeners)
  - loaded as needed (by incoming events)
  - unloaded when idle for a certain amount of time

- add this in your extension manifest file:

```json
"background" : { 
  "scripts": ["background.js"],
  "persistent": false
}
```

- use `browser.storage` to maintain information across the lifetime of your background page
- register listeners at top level
- use `browser.alarms` instead of timers (timers won't trigger if the page has been unloaded)
- don't use `getBackgroundPage()`, it won't wake up the background page if it's already been unloaded
- remove all `webRequest` listeners, as it's incompatible with non-persistent background pages
- you can check if a background page has been unloaded in Safari: go to `Develop > Web Extension Background Pages`

## declarative content blocking 

- new content-blocking API
- Expressed in JSON
- grouped into rulesets
  - javascript API to toggle rulesets on and off
- specify a ruleset in your manifest file:

```json
"declarative_net_request": {
  "rule_resources": [{ 
    "id": "my_ruleset",
    "enabled": true,
    "path": "rules.json" 
  }]
},
"permissions" [ "declarativeNetRequest" ]
```

Every rule file has four pieces:

- an unique id
- a priority - which determines the order in which the rules are applied
- an action - lets you block, allow, or upgrade the scheme of a resource
- condition - where you tell Safari where and under what conditions to run this rule
  - `regexFilter` is matched against the resource URL
  - `resourceTypes` specifies the types of resources that will be blocked (supported types: `main_frame`, `sub_frame`, `stylesheet`, `script`, `image`, `font`, `xmlhttprequest`, `ping`, `media`, `websocket`, and `other`)
  - `excludedResourceTypes` lets you specify the types that you don't want to match against
  - `domainType key` allows you to block a resource based on the relation of the domain of the resource being loaded and the domain of the document ( `first-party` load is any load where the URL has the same security origin as the document. Every other case is `thirdParty`)
  - `isUrlFilterCaseSensitive` allows you to control whether the `regexFilter` is case sensitive or not (`true` by default)

```json
{
  "id": 1, 
  "priority": 1, 
  "action": { "type": "block" },
  "condition": { 
    "regexFilter": "example.com",
    "resourceTypes": [ "script" ]
  }
}
```

## Customizing new tabs

- from iOS 14.1
- declared in manifest:

```json
"browser_url_overrides": {
  "newtab" : "fun_page.html" 
}
```
