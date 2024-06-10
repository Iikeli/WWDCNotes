# Training Text Classifiers in Create ML

Create ML now enables you to create models for Natural Language that are built on state-of-the-art techniques. Learn how these models can be easily trained and tested with the Create ML app. Gain insight into the powerful new options for transfer learning, word embeddings, and text catalogs.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/428", purpose: link, label: "Watch Video (12 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Introduction

Given a text, the [`NaturalLanguage` framework][NLDocs] classifies it.

Examples:

- Sentiment analysis (positive/negative)
- Spam detection (spam/not-spam)
- Topic classification (the text is about food, politics, or science)

## Crate ML

For testing, like for images, we can drag and drop `.txt` files, or we can simply type directly in Create ML: 
at that point every time we hit space or after not typing for a while, CreateML will run our model in the sample and classify the text.

## What's New

Text Classification has been supported since iOS 12, this year Apple is adding the support for the state of the art transfer learning.

## What is Transfer Learning?

Transfer Learning is a powerful machine learning technique where a model trained on a huge amount of data for one particular task can be repurposed for another task so that we, as developers, only need to prepare a limited amount of data and still get our customized machine learning model. 

## Tips

- Pick the best algorithm for our case (transfer learning is just one of them)

- Collect the same amount of texts for each category.
- Data consistency: if we expect our Text Classifier to mostly work on short sentences, our training data should be mostly consisting of such examples. If we expect our Text Classifier to work on short words, paragraphs, or even an entire article, we also want to make sure our training data covers these cases as well.

[NLDocs]: https://developer.apple.com/documentation/naturallanguage