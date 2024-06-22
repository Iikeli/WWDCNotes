# Drawing Classification and One-Shot Object Detection in Turi Create

Apple’s open source toolset, Turi Create, recently added tasks for Core ML model creation including Drawing Classification and One-Shot Object Detection. Learn how to quickly use these capabilities in your apps as well as new techniques for visualizing and evaluating the performance of your custom models.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/420", purpose: link, label: "Watch Video (42 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## What’s Turi Create?

Turi Create is a Python library used for creating Core ML models. It has a simple, easy to use task-oriented API.

Tasks are our term for a collection of complex, machine learning algorithms abstracted away into a simple, easy to use solution for you to build your models.

[Open source](https://github.com/apple/turicreate).

## How to create a CoreML model

Five steps:

1. Identify the task, aka what problem we’re trying to solve 
2. Collect data
3. Train a model
4. Test/evaluate the model
5. Deploy the model

Here’s the code (in python):

```python
import turicreate 

// Load data 
data = turicreate.SFrame("ActivityData.sframe")
train, test = data.random_split (0.8) 

// Create a model 
model = turicreate.activity_classifier.create(train, 

// Evaluate the model
metrics = model.evaluate(test) 

// Export for deployment 
model.export_coreml("MyActivityClassifier.mlmodel") 
```

## Task types

| ![][oldTasksImage] | ![][newTasksImage] |
| Current | New |

## One-Shot Object Detection

The one-shot object detection is like object detection, but requires much less train data.

Let’s say that we want to detect objects like cards and logos with our camera. These features are 2D features:  
One-Shot Object Detection is able to take advantage of these characteristics of regularity and consistency to dramatically reduce the amount of data needed to build an accurate model.

Object Detection normal approach:  
Take plenty of pictures, annotate all the pics with the object location and tag said object:

![][cardsImage]

One-shot approach:
One clear picture per class

![][cardsClearImage]

This works with a process called Synthetic Data Augmentation. Turi Create ships with a collection of images of the real world. The script takes the provided image and overlay onto these real-world images along with various color perturbations and distortions to simulate real world effects. The script also takes care of annotating these images.

On the session demo the Turi script has created 2000 images per class (and annotate them etc).

One-shot vs traditional approach:

![][oneVStraditionalImage]

## Drawing Classification

Drawing classification lets us use PencilKit input to train our model and recognize gestures.
In short PencilKit strokes recognition.

The Drawing Classification task can accept bitmap-based drawings, and stroke-based drawings. 

CoreML has been optimized so that we need as few as 30 drawings per class.

Turi also offers a way to explore all the tests and see the results:

![][resultsImage]

This window has three sections:

- **Overview**: gives us the overall accuracy of the model as well as the number of iterations in the model that we've trained, the Drawing Classifier model. 

- **Per Class breakdown**: breakdown of the metrics as well as a couple of example images that have been correctly been predicted by the model. 

- **Error section**: shows us all of the errors that the model has made.

[oldTasksImage]: WWDC19-420-oldTasks
[newTasksImage]: WWDC19-420-newTasks
[cardsImage]: WWDC19-420-cards
[cardsClearImage]: WWDC19-420-cardsClear
[oneVStraditionalImage]: WWDC19-420-oneVStraditional
[resultsImage]: WWDC19-420-results