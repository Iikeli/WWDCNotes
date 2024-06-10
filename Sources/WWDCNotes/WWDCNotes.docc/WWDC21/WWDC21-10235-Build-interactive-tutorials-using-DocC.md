# Build interactive tutorials using DocC

Discover how you can author immersive tutorials from scratch with DocC. We’ll demonstrate how you can bring together rich instructions, example code, and images through the DocC syntax to showcase your Swift framework in action. And we’ll go over how to create progressive training that can provide interactive learning opportunities and help people better understand use cases for your framework.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10235", purpose: link, label: "Watch Video (22 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Overview

Tutorials use a special syntax to wrap markdown in a directive that provides structure so DocC can build the complex layout and interactions featured in the tutorial.

This is a basic directive that tells DocC this text is a single step of a tutorial:

```markdown
@Step {
   Step description

   @Code(name: "file.swift" file: step_8.swift)
}
```

Along with the instruction text, this example includes a link to a Swift file that DocC will display to show readers exactly what code to write to accomplish the step

### DocC directives can be nested

In the following example, the code directive contains an image directive that provides the reader more context for the step:

```markdown
@Step {
   Step description

   @Code(name: "file.swift", file: step_8.swift) {
     @Image(source: "step_8.png")
   }
}
```

### Table of Contents

- The table of contents file starts with a `@Tutorials` directive, which contains all the elements of the page
- Inside the `@Tutorials` directive, there is an `@Intro` directive, which includes a title and a brief description of what the framework adopter will build throughout the tutorials
- You use `@Chapters` to organize tutorials into groups that make sense together
- Inside the Chapter, there are individual links to tutorials

```markdown
@Tutorials(name: "SlothCreator") {
  @Intro(title: "Meet SlothCreator") {
    Create, catalog, and care for sloths using SlothCreator. Get started with SlothCreator by building the demo app _Slothy_.
    
    @Image(source: slothcreator-intro.png, alt: "An illustration of 3 iPhones in portrait mode, displaying the UI of finding, creating, and taking care of a sloth in Slothy — the sample app that you build in this collection of tutorials.")
  }
  
  @Chapter(name: "SlothCreator Essentials") {
    @Image(source: chapter1-slothcreatorEssentials.png, alt: "A wireframe of an app interface that has an outline of a sloth and four buttons below the sloth. The buttons display the following symbols, from left to right: snowflake, fire, wind, and lightning.")
    
    Create custom sloths and edit their attributes and powers using SlothCreator.
    
    @TutorialReference(tutorial: "doc:Creating-Custom-Sloths")
  }
}
```

- DocC generates some elements of the introduction, such as the "Get Started" button and timing calculation automatically when you provide links to the tutorials in the Table of Contents page.

### Tutorial

- The Tutorial page starts with a single `@Tutorial` (note: that this time is singular) directive that contains the contents of the page
- To provide clarity along the path of building, tutorials are broken up into sections
- Sections contain steps, which instruct the adopter on what exactly they need to do to move on to the next step. Steps should be short, easy to understand, and easy to follow

```markdown
@Tutorial(time: 20) {
  @Intro(title: "Creating Custom...") {
    This tutorial guides you through...
    
    @Image(source: ...)
  }
  
  @Section(title: "Create a new...") {
    @ContentAndMedia(layout: horizontal {
      Create and configure an Xcode...

      @Image(...)
    } 

    @Steps {
      @Step { ... }
      @Step { ... }
    }
  }
}
```

- DocC can also automatically compare the current code file with the one from previous step and highlight the new portion of the code

## Creation

- Create a new tutorial by adding a new Tutorial file under the `Tutorials` folder in your documentation catalog
