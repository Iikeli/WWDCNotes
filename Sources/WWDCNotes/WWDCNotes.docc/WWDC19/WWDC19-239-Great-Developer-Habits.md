# Great Developer Habits

Successful app development requires mastering a lot of different things. Discover practices you can incorporate into your development workflow to enhance your productivity, and improve your app’s performance and stability. Learn how to improve the quality of code you write with Xcode. Gain a practical understanding of some valuable development techniques.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/239", purpose: link, label: "Watch Video (34 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Organize

### File Structure

- Xcode projects benefit from structure and organization using groups. 
- Organization makes it so much easier at a glance to see the files involved in each section of your application.
- Groups are best used to organize your project functionally in a way that logically follows how someone might interact with your application.
- Make sure that your Xcode project structure and your file system structure actually match each other. Since Xcode 9, when you create a new group inside of your project, it actually also creates a folder on disk to house the files that you place inside of that group. This means when you're looking at your project in source control, or just browsing the file system, the structure is mirrored, and this will really help you reduce confusion and missteps later on.

### Storyboards

- One storyboard for each section of the app.
- Use storyboards references.

## `.xcodeproj`

- Keeping your project file modern is a critical way to make sure that Xcode can help you out and avoids the accumulation of issues.
- Whenever prompted, or whenever a warning appears in the issue navigator, have Xcode update the project settings and update your project file to the latest format.
- Make sure you’re using the latest/new build system:you can verify the build system that your project is using by looking at the project settings found under the file menu. 
- Don’t leave unused code or commented code in the project
- Establish a zero warnings policy

### TL;DR:

- Functional organization with groups 
- Mirror project structure and file structure 
- Break apart large storyboards 
- Modernize your project file
- Throw away code scraps
- Address the root cause of warnings

## Track 

- Use source control
- Keep commits small 
- Write useful commits messages
- Branch for bugs and new features. When ready squash them together back into the main or dev batch, and use a clean and helpful commit message.

## Document 

- Two of the greatest contributors to clarity and maintainability, are code comments and documentation. 
- A good code comment explains why that code was written in the first place.
- Use descriptive names for your variables, and fully document your functions, properties, Structs and Enums

## Test 

- Even for sections of code that seem deceptively simple, it's so important to write those unit tests.
- Run unit tests before committing code
- Build a foundation for continuous integration

## Analyze 

- Use the Network Link Conditioner
- Inside of your app scheme settings, there are several sanitizers and checkers that can help you discover various issues throughout your development cycle. 
  - Address Sanitizer: monitors memory corruption and buffer overflows.
  - Thread Sanitizer: detects data races (Simulator only)
  - Undefined Behavior Sanitizer: when a program has undefined behavior, it might cause a crash, It might act in unpredictable ways, or it might act like it has no problem at all with different results at different times seemingly with no reason.
  The Undefined Behavior Sanitizer captures bugs like dividing by zero, out of range casts between floating point types, overflows, and misaligned pointers. 
  - Main Thread Checker: Ensures that we’re not performing invalid usage of appKit, UIKit, and other API's on background threads.

- Debug navigator: Monitor the Debug Gauges in the debug navigator anytime you've built and run your project. Here you can check out CPU, memory, disk, and network utilization throughout the lifecycle of your app.

- Instruments: If you need to take this even further, use instruments to run an even more in-depth analysis.

## Evaluate 

- When working with others, appreciate the opportunity to bounce ideas off of others, and get their opinions on ways to go about doing things (basically grooming).

## Decouple 

- Determine functional segments and break them out 
- Endeavor to create small, refined, reusable, and testable sections of code.
- Packages and frameworks offer an opportunity to maintain that shared code in a more centralized way.
- If your app includes extensions, your binary size will actually reduce because both your main app and your extensions can actually share that same framework. 
- Creating packages also offers the opportunity to share our efforts with the community especially with the tight integration now found in Xcode 11.
- Document your packages.

## Manage

- Use community and open source projects responsibly 
- Understand dependencies thoroughly
- Ensure that privacy is respected
- Have a plan if a dependency goes away or is no longer maintained
