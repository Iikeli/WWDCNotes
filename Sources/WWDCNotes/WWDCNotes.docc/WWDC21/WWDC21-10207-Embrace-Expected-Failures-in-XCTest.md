# Embrace Expected Failures in XCTest

Testing is a crucial part of building a great app: Great tests can help you track down important issues before release, improve your workflow, and provide a quality experience upon release. For issues that can’t be immediately resolved, however, XCTest can help provide better context around those problems with XCTExpectFailure. Learn how this API works, its strict behavior, and how to improve the signal-to-noise ratio in your tests to identify new issues more efficiently.

@Metadata {
   @TitleHeading("WWDC21")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc21/10207", purpose: link, label: "Watch Video (11 min)")

   @Contributors {
      @GitHubUser(Jeehut)
   }
}



- Why do we test? → To discover bugs before we ship and not afterwards
- Testing is an investment, so let's maximize our returns while minimizing our costs
- New failures are valuable: indicates a flaw in product, tests or dependencies
- Ideally, new failures should be fixed fast, re-appearing failures are "noisy" though
- Disabling: Test code with compile, but reduces visibility as an issue that needs resolve
- `XCTSkip`: Builds and executes, so included in test reports, but skips some useful parts
- New `XCTExpectFailure`: test executes normally, fails as "expected", but suite passes
- Allows for adding a link in `<>` to issue tracker, test report shows a bug button to get to it
- Stateful: Add `XCTExpectFailure` to the beginning of test method, marks whole method
- Scoped: Add failing code as trailing closure to `XCTExpectFailure` only ignores that part
- Use the scoped variant for shared/reused code, such as utility code
- Possible to define `XCTExpectedFailure.Options` with `issueMatcher` to control match
- E.g. `options.isEnabled` only set on a specific platform (where it's failing)
- `XCTExpectFailure` by default is `strict` and expects a test to actually fail
- For non-deterministically flickering tests, set `isStrict` to false, has convenience parameter
- Use test repetitions to debug them, see "Diagnose unreliable code with test repetitions"
