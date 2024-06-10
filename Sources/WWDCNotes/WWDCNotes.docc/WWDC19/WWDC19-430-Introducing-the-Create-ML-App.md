# Introducing the Create ML App

Bringing the power of Core ML to your app begins with one challenge. How do you create your model? The new Create ML app provides an intuitive workflow for model creation. See how to train, evaluate, test, and preview your models quickly in this easy-to-use tool. Get started with one of the many available templates handling a number of powerful machine learning tasks. Learn more about the many features for continuous model improvement and experimentation.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/430", purpose: link, label: "Watch Video (14 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Create ML

![][appIconImage]

The Create ML, Apple’s app to create new Machine Learning models, has a new workflow and two new input types.

_“It provides a really approachable way to build custom machine learning models to add to your applications.”_

> To open Create ML: launch Xcode 11, then go to `Xcode > Open Developer Tool > Create ML`. Alternatively, use spotlight and search Create ML.

## Model Types

![][modelsImage]

Activity and Sound are this year new models!

Each CoreML type offers different solutions.

## Image Model

### Image classifier

An Image Classifier can be used for categorizing images based on their contents. For example, the Art Style identifier uses a custom Image Classifier to determine the most likely movement of a piece.

![][classifyImage]
￼
### Object Detector

Lets you identify multiple objects within an image.

![][detectImage]
￼
## Sound Model

### Sound Classifier

Determines the most dominant sound within an audio stream.

![][soundImage]
￼
## Activity Model

### Activity classifier

Puts together data from the accelerometer and gyroscope to guess the type of activity the user is doing.

![][activityImage]
￼
## Text Model

### Text classifier 

Text classification can be used to label sentences, paragraphs, or even entire articles based on their contents.

We can train these for custom topic identification or categorization tasks

### Word tagger

Ideal for labeling tokens or words of interest in text. General purpose examples of this are things like tagging different parts of speech or recognizing named entities.

## Tabular Model

This is a generic model.

### Tabular Classifier

Classifiers are for categorizing samples based on their features of interest. And features can be a variety of different types such as integers, doubles, strings, so long as your target is a discrete value.

What's unique about the Tabular Classifier is it extracts away the underlying algorithm for you. And identifies the best multiple classifiers for your data.

### Tabular Regressor

A model that will predict a numeric value, such as a rating or a score.

### Recommender

Allows us to recommend content based on user behavior.

The Recommender can be trained on user-item interactions with or without ratings.

## Create ML Workflow

Five steps:

1. choose your domain/model (aka your input type among the five mentioned above).
2. based on the domain, the app will offer you different model types. 
3. Input
4. Training
5. Output

Create ML will always display analytics during your progress (with user-friendly charts) to make you easily understand the accuracy of your model.

In Apple words: _“And with additions like metrics visualization, live progress, and interactive preview, Create ML app sets the bar for a great model training experience.”_

The testing can be done by simply dragging and dropping new data (of all types, based on what model we’re training .

The session goes on and shows how to train a flower recognizer, it skips some steps but it overall looks very promising and simple to implement. Most importantly, the `.mlmodel` produced by Create ML is incredibly small (way less than 100KB from a collection of several images).

> It goes without saying, but all is mentioned here run on devices in their Neural Engine: no cpu and gpu wasted on these (only if you have a A12 or S4 device, earlier devices will run CoreML in the GPU instead).


[appIconImage]: WWDC19-430-appIcon
[modelsImage]: WWDC19-430-models
[classifyImage]: WWDC19-430-classify
[detectImage]: WWDC19-430-detect
[soundImage]: WWDC19-430-sound
[activityImage]: WWDC19-430-activity