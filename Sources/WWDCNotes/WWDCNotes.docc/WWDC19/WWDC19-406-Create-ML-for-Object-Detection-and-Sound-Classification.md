# Create ML for Object Detection and Sound Classification

Create ML enables you to create, evaluate, and test powerful, production-class Core ML models. See how easy it is to create your own Object Detection and Sound Classification models for use in your apps. Learn strategies for balancing your training data to achieve great model accuracy.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/406", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(Blackjacx)
   }
}



- **Object detection** 
  - Download existing object detectors for broad categories, but train your own models for specific subtle differences
  - **Image classification** describes the complete image, but **Object detection** can identify and locate multiple objects
  - For training object detection we need to annotate labeled regions (label, center (x,y), width, height) and store that as JSON file in the same folder as the images. 
  - Demo: use **Create ML** to build a model for recognizing the numbers on the top of (multiple) dice
    - Drag folder to training data and Create ML will do some basic checking/validation
    - Training object detection takes much longer than image classification
    - Visualizing training and results
    - Test with new images directly in Create ML (use Continuity to access the iPhone camera)
    - Export mlmodel-file to use in your app

- 
  - Considerations:
    - Balanced number of images in each class
    - 30+ images per class
    - Real-world data: multiple angles, backgrounds, lighting conditions, different _other_ objects
    - A single class can be enough (model learns to locate this class in images)

- 
  - Use **Vision** framework to integrate into app

- **Sound Classification** model training in Create ML
  - Identify the source of the sound (e.g. guitar vs. drums or nature vs. city) or identify properties (laugh vs. cry)
  - Demo: use **Create ML** to build a model for classifying musical instruments from sound
    - Sound files from different instruments in different folders
    - Drag into Create ML, automatically separated into Training and Validation
    - Test the model in Create ML on files or live on microphone
    - Export mlmodel-file to use in your app

- 
  - Considerations
    - Add background noise as a separate category
    - One class per file (split if necessary)
    - Real-world audio environments
    - **AVAudioSessionMode** selection

- 
  - Integration
    - New framework: **Sound Analysis** for automatic channel mapping, sample rate conversion and audio buffering before applying the model
    - Results are in block size (e.g. one second) overlapping by 50% (default)
