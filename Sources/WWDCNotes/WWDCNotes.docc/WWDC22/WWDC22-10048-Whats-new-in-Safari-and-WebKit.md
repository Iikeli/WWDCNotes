# What's new in Safari and WebKit

Explore the latest features in Safari and WebKit and learn how you can make better and more powerful websites. We’ll take you on a tour through the latest updates to HTML, CSS enhancements, Web Inspector tooling, Web APIs, and more.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10048", purpose: link, label: "Watch Video (31 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## New HTML & CSS features

- [`dialog`][dialog] tag
- [`::backdrop`][backdrop] pseudo-element
- inert
- images lazy loading

```html
<img src="images/shirt.jpg" loading="lazy"
     alt="a brown polo shirt"
     width="500" height="600">
```

- container queries support from Safari 16

```css
.container {
  container-type: inline-size;
  container-name: clothing-card;
}

.content {
  display: grid;
  grid-template-rows: 1fr;
  gap: 1rem;
}

/* 👇🏻 if the clothing card component is in a container that's wider than 250 pixels, */
@container clothing-card (width > 250px) { 
  .content {
    grid-template-columns: 1fr 1fr;
  }
  /* additional layout code */
}
```

- cascade layer support

```css
/* Author Styles - Layer A */
@layer utilities {
  div {
    background-color: red;
  }
}

/* Author Styles - Layer B */
@layer customizations {
  div {
    background-color: teal;
  }
}

/* Author Styles - Layer C */
@layer userDefaults {
  div {
    background-color: yellow;
  }
}
```

- `has()` support
- new viewport units:
  - `svw`, `lvw`, `dvw` - smallest/largest/dynamic view port width
  - `svh`, `lvh`, `dvh` - smallest/largest/dynamic view port height
  - `svb`, `lvb`, `dvb` - block
  - `svi`, `lvi`, `dvi` - inline
  - `svmin`, `lvmin`, `dvmin` 
  - `svmax`, `lvmax`, `dvmax`

- [`offset-path`][offset-path-property] for animation
- `focus-visible` pseudo-class
- `accent-color`
- `font-palette`
- `text-decoration-skip-ink`
- `ic unit`
- Subgrid

```css
/* Grid to layout cards */
main {
  display: grid;
  grid-template-columns: 
    repeat(auto-fit, minmax(225px, 1fr));
  gap: 1rem;
}

/* Grid to layout each card’s content */
article {
  display: grid;
  grid-row: span 5;
  grid-template-rows: subgrid; // 👈🏻
}
```

- dropped webkit prefixes from:
  - `backface-visibility`
  - `print-color-adjust`
  - `text-align: match-parent`
  - `mask` (renamed)
  - `text-combine-upright` (renamed)
  - `appearance` (renamed)

## Web Inspector

- new Flexbox inspector
- new alignment editor
- new justify-content editor
- new CSS fuzzy autocompletion
- developer tool extensions support

## Web API

- support for Web Push
  - Safari 16 + macOS Ventura
  - iOS and iPadOS next year

- new web app manifest improvements
  - you can define the icon that's used when people save your web app to the Home Screen

```json
// Manifest file 

"icons": [
 {
   "src": "orange-icon.png",
    "sizes": "120x120",
    "type": "image/png"
  }
]
```

> ensure that there is no `apple-touch-icon` defined in the HTML head

- Broadcast Channel
  - post a message and sync that state to any other open tabs or windows

```js
broadcastChannel.postMessage("Item is unavailable");
```

- File System Access API

```js
// Accessing the origin private file system
const root = await navigator.storage.getDirectory();

// Create a file named Draft.txt under root directory
const draftHandle = await root.getFileHandle("Draft.txt", { "create": true });

// Access and read an existing file
const existingHandle = await root.getFileHandle("Draft.txt");
const existingFile = await existingHandle.getFile();
```

[dialog]: https://www.w3schools.com/tags/tag_dialog.asp
[backdrop]: https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop
[offset-path-property]: https://www.w3.org/TR/motion-1/#offset-path-property
