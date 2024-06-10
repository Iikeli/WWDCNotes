# Fix failures faster with Xcode test reports

Discover how you can find, debug, and fix test failures faster with the test report in Xcode and Xcode Cloud. Learn how Xcode identifies failure patterns to help you find the right place to start investigating. We’ll also show you how to use the UI automation explorer and video recordings to understand the events that led up to your UI test failure.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10175", purpose: link, label: "Watch Video (13 min)")

   @Contributors {
      @GitHubUser(RamitSharma991)
   }
}



- Test runs range from a single test you're running while working on a piece of code to an entire suite with thousands of tests running in CI.
- Test report is where you go to view results for test runs that happen locally,in Xcode Cloud, or on another machine.
- The test report organizes your test results in a way that helps you understand: 
  - the health of your project
  - identify problem areas
  - ultimately fix failures faster

## Structuring tests

* Test methods are the individual tests or methods which validate your source code and produce test results.
* Test classes are groups of test methods and are usually grouped based on the area that's being tested.
* Test bundles are composed of one or more test classes.
* Each bundle houses a single type of test, either Unit or UI. 

### Unit vs UI Tests

* Unit tests help verify a single piece of code, generally a function. 
* Unit tests are short, simple, and run very quickly. 
* UI tests observe the user-facing behavior of your app.
* UI tests make sure your app truly does what you expect it to. 
* Test bundle contains UI tests.
* At the highest level, there are test plans. 
* Test plans contain one or more test bundles, which means a test plan can contain both Unit and UI tests. 
* Test plans, helps setting up configurations to efficiently run your tests under several conditions.

### Configurations

* Important aspect of Test plans.
* tells Xcode how to set up the runtime environment for tests.
* you can test your app in varying Languages and Locations. 
* Test with code coverage to keep track of the quality and coverage of your code as you continue to develop. 
* You can even set up your tests to run many times ensuring all elements of your app are reliably working. 

### Run Destinations

* are the devices where your tests run. 
* When running tests in Xcode's IDE, you can select a single run destination. 
* With Xcode Cloud and xcodebuild command, your tests can have multiple run destinations.

### Tests, configurations, and run destinations working

- The test plan runs on each device once with a configuration enabled. 
- Each method will exit with a test result status, either passed, failed, skipped, or expected failure.
- Xcode runs the full test plan once for each configuration and run destination, resulting in a whole matrix filled with results.
- Result is produced for every test method, configuration, and run destination combination.
- This individual instance is called a test method run.
- Test runs range from a single test you're running while working on a piece of code to an entire suite with thousands of tests running multiple configurations on multiple destinations.

## Explore the test report

* The new test report gives you tools to help you understand your test run, regardless of the number of tests.
* provides a high level summary of your test run, so you can see the big picture before digging into the details. 
* Highlights important patterns, to quickely start investigating.
* Gives you a single place to see test activity, failure information, screenshots, and more.
* improved UI test debugging tools, giving richer failure information.
* The test summary gives an overall understanding of what happens in a test run.
* Easily understandable testing environment. 
* explore any notable patterns found in during test results using Insights. Insights:
  * are the patterns Xcode found while analyzing results across all configurations and run destinations.
  * groups results based on certain criteria.
  * two types of Insights:
    * Common Failure Patterns: groups tests based on similar failure messages.
    * Longest Test Runs: clues you in on which tests in your test bundles are taking longer than the others.
* Within the test section, you can check tests performed during a run and get more details about the test plan. 
* Check special traits that the test plan has, like test repetitions or performance metrics. 
* With  heat map, check tests on each device and configuration.
* Colors and test result counts helps to understand how this run did when compared to the others. 
* if there’re any test failures, there’s quick access to them on the test summary. 
* If there’s a particular failure use this section to start investigating.
* The test report has a ton of awesome new features.
* The activities tab contains three major sections: 
* Test activity: 
  * The test activity lays out the test in a timeline format, where the top-most row is the start of the test.
  * the bottom row is the end, and each row in between is an event which took place in the test. 
* Automation explorer:
  * find moments of video playback related to the selected test activity. This allows a full replay of test. 
* Scrubber: 
    * a linear representation of test run. 
    * locate test events, like taps, swipes, and clicks. 
    * highlights when the device-under-test changes orientation. 
    * failure icon above the scrubber notes where in the test the failure occurred.
* the test report has a test debugging experience that is interactive.
* Clicking on an event in the activities pane updates the automation explorer with the corresponding frame from video playback, to visually understand what's happening at each moment of the test.
* It also moves the scrubber to the right spot, to watch where events are taking place in relation to the full test run.  
* Available on both Xcode and Xcode Cloud. 
