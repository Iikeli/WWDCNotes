# Elevate your DocC documentation in Xcode

Great documentation can help people effectively and easily adopt your Swift framework. Discover how you can create rich, conceptual articles to accompany your API. You’ll learn best practices for writing articles, including how to structure your documentation, and find out how to create automatically managed links that connect your docs together.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10167", purpose: link, label: "Watch Video (17 min)")

   @Contributors {
      @GitHubUser(Jeehut)
      @GitHubUser(zntfdr)
   }
}



## Page types

Documentation Catalog offers three page types:

- Reference - concise, in-depth information about individual APIs in your library. E.g., descriptions, code snippets, relationships
- Articles - free-form content, give the big picture of how a framework works and explaining how to complete a specific task
- Tutorials - step-by-step walk-through of a project that uses your framework

All page types follow the Markdown format.

### Top-level article

Top-level articles provide:

- a concise summary sentence
- an overview with content like images and code snippets
- a topics section with a few symbols you want to highlight

### Task article

- provide actionable first steps on how to get started with your library
- similar to top-level article, but focuses on teaching how to use parts of the library

### Adding articles to the documentation

- set up my project with a documentation catalog (`New File > Documentation Catalog`)
- a documentation catalog is a `.docc` file in the Xcode navigator that contains all my documentation files
- a new documentation catalog already comes with a top-level article by default

## Organization

- assets
  - to add assets, put them in the `Resources` under your documentation catalog
  - Image best practices to use `@2x` PNG files, add also `~dark` variant (if needed)
  - Assets can be added to `Resources` folder of the catalog, include via `![title](x.png)`
  - when referring to an image, only   the name + the format, e.g. `sloth.png`, Xcode will automatically select the correct appearance and scale

- articles
  - once added in the catalog, the DocC compiler automatically adds a `Topics` section to each page with all of the framework’s documentation.
  - to customize this, we can fill the `## Topics` section directly in the page
  - use the `<doc:...>` URL scheme to refer to articles
  - use double ``` ``...`` ``` back ticks to link to a symbol 
  - group types into groups, sort them from easy to complex

```markdown
## Topics

### Essentials

- <doc:GettingStarted>
- <doc:/tutorials/SlothCreator>
- ``LibraryType``
```

## Extensions

Extension files extend the documentation of each symbol by associating an extra (optional) documentation file

- this file links to a specific symbol by linking the extension file title

```markdown
# ``LibraryName/LibraryType``

## Topics
...
```

- DocC will automatically add the extension file content to the rest of the symbol documentation.
- Documentation can be exported so browsing in Xcode without package is possible
