# Introduction to Attributed Strings for iOS

Text is an essential part of your application's interface. Get an introduction to the concepts behind manipulating and drawing attributed strings on iOS. Learn how UIKit has adopted attributed strings to make creating text effects even easier.

@Metadata {
   @TitleHeading("WWDC12")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc12/222", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



This is the year `NSAttributedString` was introduced:  
previously all our UI elements like buttons, textViews etc could only accept attributes for the whole string.

No char no attributes.

If we’re adding a string to an empty `NSMutableAttributedString`, this string will have no attributes attached.

If we’re inserting a string to a non-empty `NSMutableAttributedString`, the string will inherit the attributes of the char which correspond to the left limit of the insertion range.

The same is true if we’re substituting: each char **won’t** get the attributes of the char that they replace.

You can replace the characters of an attributed string while preserving the attributes as in the example below:

```objc
NSMutableAttributedString *attrString; 
NSLocale *locale = [NSLocale currentLocale]; 

[attrString enumerateAttributesInRange:NSMakeRange(0, [attrString length]) 
  options:0 
  usingBlock:^(NSDictionary *attrs, NSRange range, BOOL *stop) {
    NSString *string = [(attrString string] substringWithRange: range]; 

    [attrString replaceCharactersInRange:range 
                withString: (string uppercaseStringWithLocale:locale]];
}]; 
```

Note how we need to enumerate the attributes so we have a block call for each range where the attributes change.  
If we just replace everything in one shot, as said above, all the characters of the new string will just inherit the attributes of the first char that we’re replacing.

We have paragraph styles! See example below (the !!! symbol is not part of the label):

![][remindersImage]

To do this we need a `NSMutableParagraphStyle` with `setFirstLineHeadIndent` and then add the `NSMutableParagraphStyle` to our attributed string as an attribute.

This is list of paragraph styles (as of WWDC 2012):

- Line break mode
- Alignment
- Spacings
- Indentation
- Line height control
- Hyphenation
- Base writing direction

If we assign an attributedString to a `UILabel`, then we can still use the `UILabel` `textColor`, `textAlignment`, `lineBreakMode` etc, but bare in mind that those change will change the whole string (the rest of the attributes will remain unchanged).

[replaceImage]: ../../../images/notes/wwdc12/222/replace.png
[paragraphImage]: ../../../images/notes/wwdc12/222/paragraph.png
[remindersImage]: WWDC12-222-reminders
