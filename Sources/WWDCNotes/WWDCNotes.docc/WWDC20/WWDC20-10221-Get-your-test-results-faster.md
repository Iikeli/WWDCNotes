# Get your test results faster

Improve your testing suite to speed up your feedback loop and get fixes in faster. Learn more about the latest improvements to testing in Xcode, including how to leverage test plans, Xcodebuild updates, and APIs to eliminate never-ending and badly-behaved tests. We’ll explore Test Timeouts and Execution Time Allowances in XCTest, examine device parallelization, and detail recommended practices for balancing performance with clear fault localization.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10221", purpose: link, label: "Watch Video (16 min)")

   @Contributors {
      @GitHubUser(skhillon)
   }
}



## The Testing Feedback Loop
Either this loop does not make you confident enough and you need to write more tests, or you are confident and you can move to the next task.

![][testing_feedback_loop]

The speed of this feedback loop is critical to your development speed.

## Overview
1. Ensuring you always get feedback
2. Getting even faster feedback

### Real world example
It's Monday and you're looking at CI results from Friday, but your test suite never actually finished. If you don't investigate this problem, you'll lose confidence in your tests and therefore the application code that you're shipping.

## 1. Ensuring you always get feedback
In the example above, our tests hung and we never got results, so we can't interpret them. The CI output below just says that the CI job was cancelled, which isn't actionable.

![][hanging_ci_test]

We need to figure out what went wrong in the first place. Some possible culprits are:

- Deadlock
- Extremely slow progress
- Bad timeout values in application code
- Too much work on your main thread

### Execution Time Allowance
Xcode 12 introduces an opt-in feature that allows you to enforce a time limit on each individual test. When a test exceeds this limit, Xcode will:

1. Capture a spindump
2. Kill the test that hung
3. Restart the test runner so that the rest of the suite can execute

#### What is a spindump?
- Shows you which functions each thread is spending the most time in
- You can capture a spindump using the `spindump` utility in the terminal or Activity Monitor on macOS.

#### Customization
- By default, a test gets 10 minutes.
- If you need more time for all tests, you can customize the default allowance in the test plan.
- If you need more time for a specific test/test class, you can use the `executionTimeAllowance` API.

```swift
// Give an individual test more time

class XCTestCase: XCTest {
  var executionTimeAllowance: TimeInterval  // rounded up to nearest minute
}
```

For a demo of how to enable this in Xcode, jump to 6:54 in the video. Here's some helpful slides:

![][customizing_default_allowance]

![][time_precedence]

![][maximum_allowance]

#### New test results
The test reporter now shows that a single test exceeded the time limit.

![][new_test_report]

There is also an attached spindump, which you can double-click to open. Spindumps are generally broken into 2 sections: a preamble with metadata, and then a series of stack traces for each thread in the process that was sampled.

The timed-out test should be somewhere in the spindump, so you can CMD+F for the test name in the file. The output shows which functions are being called by the test, both private and public. The output also shows the test acquiring a lock and then waiting, which suggests an issue in the helper method.

In the image below, the helper method `performGETRequest` is acquiring the same lock as the method under test, `fetchSynchronouslyFromServer`.

![][double_lock]

Deleting the lock acquisition code in the helper method fixes the hang.

### Recommendations for Execution Time Allowance
- Use `executionTimeAllowance` to guard against test hangs.
- Use `XCTest`'s performance APIs to detect performance regressions in your application code.
- Use Instruments to profile your application's performance.

## 2. Getting Even Faster Feedback with Parallel Tests
You can perform distributed testing on parallel devices via `xcodebuild`.

Here's a sample test report, which took about 13 minutes to run. Some individual tests took minutes to run.

![][sample_long_test_report]

The fact that some individual tests took much longer is a hint that we might want parallel testing.

Non-distributed testing runs every test serially, one after the other.

![][non_distributed_testing]

You can speed up your tests with parallel distributed testing. In this case, `xcodebuild` will distribute tests to each run destination by class. Each device then runs a single test class at a time.

![][parallel_distributed_testing]

**Important**: The allocation of test classes to run destinations is non-deterministic. If you're testing logic that is device or OS-specific, this can lead to unexpected failures or skipped tests.

Starting with Xcode 12, the bottom row is now also supported.

![][parallel_distributed_support]

To run using `xcodebuild`, ensure that you have the `-parallel-testing-enabled YES` and `-parallelize-tests-among-destinations` flags set.

**With just 2 devices, you can get a 30% speedup using Parallel Distributed Testing**.

### Recommendations
- Use all of the same kinds of devices and OS versions per test suite run. This avoids difficult-to-reproduce test failures because Xcode non-deterministically assigns test classes to devices.
- If using different devices and OS versions, perfer running tests that are device and OS-agnostic.
- To intentionally test against more devices and OS versions (e.g. testing iOS 13 and iOS 14), use Parallel Destination Testing. This performs non-distributed testing, but on multiple devices at the same time.

## Wrap up
- Use Execution Time Allowances to ensure your tests always complete running.
- Use spindumps for diagnosing application stalls and hangs.
- Use Parallel Distributed Testing to speed up your tests.
- Use Parallel Destination Testing to simultaneously run your tests on more OS versions and devices.

[testing_feedback_loop]: testing_feedback_loop.png

[hanging_ci_test]: hanging_ci_test.png

[new_test_report]: new_test_report.png

[double_lock]: double_lock.png

[customizing_default_allowance]: customizing_default_allowance.png

[time_precedence]: time_precedence.png

[maximum_allowance]: maximum_allowance.png

[sample_long_test_report]: sample_long_test_report.png

[non_distributed_testing]: non_distributed_testing.png

[parallel_distributed_testing]: parallel_distributed_testing.png

[parallel_distributed_support]: parallel_distributed_support.png
