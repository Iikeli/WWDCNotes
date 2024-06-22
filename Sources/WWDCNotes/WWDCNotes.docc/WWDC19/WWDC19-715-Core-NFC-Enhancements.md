# Core NFC Enhancements

Learn how easy it is to add support for NFC in your app and take advantage of the newest capabilities such as NDEF writing and support for widely adopted native tag protocols.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/715", purpose: link, label: "Watch Video (30 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



With iOS 13 any iPhone from the iPhone 7 lineup and later can both read and write NDEF NFC Tags (**N**FC **D**ata **E**xchange **F**ormat).

We need to declare in the entitlements which tags we want to be able to read (basically a prefix of the tag identifier).

To write NDEF tags, we use the [`NFCNDEFReaderSession`][ndefDocs] and, after detecting the tag, we can read/write/lock it:

```swift
// New NFCNDEFReaderSessionDelegate method to receive NDEF tag objects 

optional func readerSession(_ session: NFCNDEFReaderSession, didDetect tags: [NFCNDEFTag])

// NDEF tag protocol 

var isAvailable: Bool { get } 
func queryNDEFStatus(completionHandler: @escaping (NFCNDEFStatus, Int, Error?) -> Void)
func readNDEF(completionHandler: @escaping (NFCNDEFMessage?, Error?) -> Void)
func writeNDEF(_ ndefMessage: NFCNDEFMessage, completionHandler: @escaping (Error?) -> Void)
func writeLock(completionHandler: @escaping (Error?) -> Void) 
```

The sessions come with sample code to [read][read] and [write][write] NFC tags.

[ndefDocs]: https://developer.apple.com/documentation/corenfc/nfcndefreadersession
[read]: https://developer.apple.com/documentation/corenfc/building_an_nfc_tag-reader_app
[write]: https://developer.apple.com/documentation/corenfc/creating_nfc_tags_from_your_iphone