# Advances in Natural Language Framework

Natural Language is a framework designed to provide high-performance, on-device APIs for natural language processing tasks across all Apple platforms. Learn about the addition of Sentiment Analysis and Text Catalog support in the framework. Gain a deeper understanding of transfer learning for text-based models and the new support for Word Embeddings which can power great search experiences in your app.

@Metadata {
   @TitleHeading("WWDC19")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc19/232", purpose: link, label: "Watch Video (39 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



Two UX concerns when thinking about natural language :

- output of natural language text
- User input of text (ie. search input)

iOS 12 Natural language added text classification and entity tagging

From iOS 13 there’s a new category of Sentiment analysis:

- The output is value from -1 to 1 represent a confidence score
- This analysis is extremely performant (it uses a hardware-accelerated neural network).
- The sentiment analysis is around the positivity or negativity of the sentence
- Support for seven languages

## Word Tagging 

Definition: given a sequence of words (a sentence), aka a sequence of tokens in NPL, we would like to assign a label to every token in this sequence.

![][timothyImage]

- in iOS 13, Word Tagging has gotten improved support with ‘Catalogs’.
- New this year we can pass our own dictionary (text catalogs) in word tagging to make NPL even more powerful.
- This is done via `MLGazeteer`: `MLGazetteer` and its associated APIs let you create and “Text Catalogs”.

```swift
let entities = [
  "Italian Cheese": ["Parmigiano-Reggiano", "Pecorino Romano", "Montasio", ...],
  "Mexican Cheese": ["Cotija", "Añejo", "Chihuahua", "Queso de cuajo", ...],
  "American Cheese": ["Monterey Jack", "Colby cheese", "Colorado Blackie", ...],
  "Greek Cheese": ["Feta", "Kasseri", "Manouri", "Kefalotyri", ...], 
  ...
] 
```

- You can leverage tons of data and create a model that’s super compressed. Ex. All the named places and persons in Wikipedia (2.5 million entries) take up on 2.5 MB on disk!
- It’s like language classification models similar to image classification in the Vision framework but for language
- Our gazetteer has priority over the default (iOS’s) one. Usage:

```swift
import NaturalLanguage 

let gazetteer = try! NLGazetteer(contents0f: url)
let tagger = NLTagger (tagSchemes: [.nameTypeOrLexicalClass]) 
tagger.setGazetteers([gazetteer], for: .nameTypeOrLexicalClass) 
```

## Word Embedding

### Definition

Vector representation of words.  

Basically this is the same concepts behind the Image Similarity above, similar words (sneakers and flip flops for example) will be clustered together.

Mapping of a string in continuous sequence of numbers. The vectors represent a score of association. E.g. thunderstorm -> sky, clouds, cloudy

![][embeddingImage]

### Word Embedding in iOS 13

We can also have our own Word Embedding: maybe for a particular domain or for a language not supported by default.

This also supports other standards (so we don’t need to create our own) as:

- Word2vec
- GloVe
- fasttext

- Apple uses this technique combined with Image classification to produce search results in the new iOS 13 photos app: if we search for thunderstorm the photos app will suggest results for sky too

- API supports the querying the ‘distance’ between two words, nearest neighbors etc.
- The default models supports 7 languages. expect more in future iOS releases.
- Custom Word Embedding can be more domain specific and more sophisticated knowledge

## Transfer Learning for Text Classification 

- This allows for smaller sets of data for training and improved accuracy For example, when your training data doesn’t include the words used by the user.
- Transfer of learning notices the meaning of words and the context in which they are used. Using this knowledge, ML infers the meaning  for text classification
- Limited language support (static vs dynamic)



[timothyImage]: WWDC19-232-timothy
[embeddingImage]: WWDC19-232-embedding
[Image]: ../../../images/notes/wwdc19/232/.png