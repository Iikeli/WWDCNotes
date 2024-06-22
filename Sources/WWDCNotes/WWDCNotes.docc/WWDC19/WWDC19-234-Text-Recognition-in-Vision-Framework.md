# Text Recognition in Vision Framework

Document Camera and Text Recognition features in Vision Framework enable you to extract text data from images. Learn how to leverage this built-in machine learning technology in your app. Gain a deeper understanding of the differences between fast versus accurate processing as well as character-based versus language-based recognition.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/234", purpose: link, label: "Watch Video (38 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



New `VNRecognizeTestRequest`: this object turns an image into text.

## How it works

Two possibilities/ways:

- Fast
  - detects characters
  - recognizes characters (with a small model)
  - (Optional) Runs detection into a NLP process (Neuro-Linguistic Programming)

- Accurate
  - Uses a Neural Network to find the text (in terms of sentences placement in the image)
  - Uses another Neural Network to read the text
  - (Optional) Runs detection into a NLP process (Neuro-Linguistic Programming)

Fast is...much faster (on a small note picture, ~0.25s vs 2s), however is less accurate (the more standard the text is, the better results).

Accurate is slower but much accurate, especially when “special” fonts are used.

## Other differences

![][Image]

Regarding the **NLP Processing phase**:

- Corrects typical recognition errors (zeroes to letter “o” for example)
- It can get in the way for codes (c001 can become cool)
- Increases the processing time and memory budget

Once recognized the text, we can also query for bounding box in the images for recognized text (for example, to highlight one recognized word).

When using the camera, we can also have an on-screen “region of interest” (ROI), which helps with the performance by, instead of scanning the whole camera frame, just that region. We set what this region is.

## Best practices

If you know the language of the document to be scanned, set it.
If you know the format/lexicon of what you want to scan, pass it to the Vision recognizer (like the pattern for a phone number: `+1 (0) xxx - xxx xxxx`) .
This also helps the NLP process as the result will match the passed lexicon.

If the text is big, you can set the minimum text height, which is a number from 0 to 1, where 0.3 for example is 30% of the camera image. This helps with performance (faster scanning) and lower memory usage.

If the textRecognition is not the core experience of your product, you can set it to run only on the CPU to give more gpu/neural network for other core experiences (like ARKit)

We also have available a progress handler that we can use to tell the user what it the state of the scan:
textRecognitionRequest.progressHandler = …

We can cancel a recognition if no longer needed via `textRecognitionRequest.cancel()`

## Processing Results

- Compare spatial info (position, scale, rotation)
- Use `NSDataDetector`s for addresses, URLs

[Image]: WWDC19-234-table