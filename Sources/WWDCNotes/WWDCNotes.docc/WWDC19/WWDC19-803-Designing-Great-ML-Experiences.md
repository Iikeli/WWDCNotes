# Designing Great ML Experiences

Machine learning enables new experiences that understand what we say, suggest things that we may love, and allow us to express ourselves in new, rich ways. Machine learning can make existing experiences better by automating mundane tasks and improving the accuracy and speed of interactions. Learn how to incorporate ML experiences into your apps, and gain practical approaches to designing user interfaces that feel effortlessly helpful.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/803", purpose: link, label: "Watch Video (57 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



This is a people/tech talk on how to design Machine learning experiences. Some of the things listed here automatically translate to code, some are more heuristic.

Refer to the Apple’s [ML guideline here](https://developer.apple.com/design/human-interface-guidelines/machine-learning/overview/introduction/).

## Usage

Apple uses machine learning in ways that you may not expect, subtle ways that improve your experience with your devices: AirPods, FaceID, quick type, they all use ML.

A few examples:

- In the keyboard, Apple increases or decreases the target tap area of the keyboard buttons based on the word you're most likely to type.
- In Photos, Machine learning helps people create albums, edit photos and search for specific memories.
- Searching for pictures based on what's in a picture (e.g. dog) changes how we engage with our memories. The search even suggest word completion, suggest categories to search in.
- The new Apple watches use ML to detect heart problems.

## Why Machine Learning

Looking at categorizing pictures of dogs for example, functional programming would require plenty of different logic based on the breed etc, different photos angle etc, also you’d need to avoid making wrong detections (like a hyena, which is similar to a dog).
These are experiences that we want to create where we can't just tell the computer what to do. And for many of these experiences, we can use machine learning to teach a computer what to do.

In Machine Learning the function that we give input to get an output is called a model.

To design a great machine learning experience, we have to design both how it works and how it looks and feels. We have to design both the **model** and the **interface**.

### Model

We consider the model as two parts:

- Data: What we use to teach the computer. 
- Metrics: How we evaluate the data.

### Interface

The ML interface can be split in two parts:

- Inputs: How people interact with the model and, if appropriate, provide input to improve the model.
- Outputs: What the model outputs and how those model's outputs are presented to people.

### (Model’s) Data

All the examples that we use for training the model are called Data. 

If we need to add a new class, we need more data.
If we need to improve one class, we need more data.

Data determines the behavior of the model.

It’s important to collect data intentionally:  
if you don't have data that captures an important scenario, it's unlikely that your model is going to work well in that scenario. 

Always think and cover all the possible scenarios (detecting people?  Do not use only white people in the train data, do not use only ID photos, use photos from different lighting conditions and angles).

### (Model’s) Metrics

Metrics are how we define if an ML experience is good.

Even if the metrics numbers are good, this doesn’t mean that the user experience will be good. Always try the app in real scenarios, get customer feedback, etc.

The Metrics might evaluate different things that what makes a good experience.

Always evolve your metrics based on your product.

### (Interface’s) Outputs

Different sets of outputs:

#### Multiple Options

Allows to present multiple options to people.

For example:

- In maps the app offers multiple routes, and the proposed routes are distinct by attributions (fastest, alternate, ..).
- The Siri watch face does the same: it recommends things based on location, time, the user routine etc. Even better, the watch face learns more the more the user uses it, so that the first recommended option is always the one the user expects/wants.

#### Attributions

Explanations that help people understand more about how your app makes decisions.

For example:

- We saw an example above with the Map route above.
- On the App Store, the app recommends you other apps based on your past behaviour (and it shows it to the user with a text like “Recommended based on apps you downloaded”)

The attributions shouldn’t assume anything: if a user downloads a cooking app, don’t say “we recommend this because you’re into cooking“, you should say “we recommend this because you download Coocking app 123“. 

In short, relate to objects facts, not subjective tastes. Cite data sources.

#### Confidence

Measurement of certainty for an output. 

Don’t show the confidence in your app unless your app is showing statistical predictions such as weather, sports results or election predictions, it might not work in other situations.

Use more human language/images instead.

Alternatively, provide ranges of confidence. 

For example when estimating arrival times, give a time range instead of saying 85% possibility to arrive home at 6pm.

If the model confidence is low, ask for the user help. 

For example, you can ask the user to identify if the person the model think is PersonA, is really personA or not. 

#### Limitations

Occurs when there's a mismatch between people's mental model of a feature and what the feature can actually do. 

For example in Memoji the app will alert users when there’s low lighting condition, the face is too far away etc.

Know your model limitation and promptly make sure that your user knows about it when it encounters one.
Explaining the limitations when they happen to help people learn to avoid these situations in the future.

### (Interface’s) Inputs

#### Calibration

Helps you get essential information for someone to engage in your experience.

Calibration can be done by collecting data of the user surroundings or biometric data.

HomeCourt does this by asking the user to make a throw.

FaceID does this by asking the user to rotate the face on activation.

Try to be quick and effortless.

Ask only for essential information.

Introduce, guide, and confirm the calibration (faceID does this nicely)

Allow people to update their information/calibration if possible/needed.

#### Implicit Feedback

Allows you to collect important information from interactions someone has with your experience.

For example, faceID calibrates itself every time the user uses faceID.

Siri does this in the search screen of the iPhone and in the Siri watch face.

Strive to identify people’s intention
Make interactions more accurate and delightful 

#### Explicit Feedback

Allows you to collect information but this time by asking specific questions about the results you're showing. 

Basically what Amazon/e-commerces do: they offer you things based on past purchases, and they offer you to say “I’m not interested in this” “See less of this“ “hide suggestion”.

Prioritize negative feedback over positive: for positive feedback look at the user behavior (as in, if he bookmark the item, purchase the item etc).

Act immediately and persistently on user feedback: hide suggested items and stop suggesting those.

#### Corrections

Allow people to fix a mistake a model has made by using familiar interfaces. 

When writing something, iOS sometimes suggests corrections here and there, but sometimes we want a special name instead of a word. Think Angie vs. angle. If the user correct the word to Angie, iOS learns this and will even suggest this next time.

The photos.app does the same when we start a crop of a picture: initially the app will automatically crop and rotate for us, however it doesn’t commit the result, letting the user to adjust (nee, correct) the crop/rotation.

Corrections are, therefore, an amazing pattern to optimize your results without feeling like extra work.