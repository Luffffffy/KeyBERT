## **Installation**
Installation can be done using [pypi](https://pypi.org/project/keybert/):

```
pip install keybert
```

You may want to install more depending on the transformers and language backends that you will be using. The possible installations are:

```
pip install keybert[flair]
pip install keybert[gensim]
pip install keybert[spacy]
pip install keybert[use]
```

To install all backends:

```
pip install keybert[all]
```

## **Usage**

The most minimal example can be seen below for the extraction of keywords:
```python
from keybert import KeyBERT

doc = """
         Supervised learning is the machine learning task of learning a function that
         maps an input to an output based on example input-output pairs.[1] It infers a
         function from labeled training data consisting of a set of training examples.[2]
         In supervised learning, each example is a pair consisting of an input object
         (typically a vector) and a desired output value (also called the supervisory signal). 
         A supervised learning algorithm analyzes the training data and produces an inferred function, 
         which can be used for mapping new examples. An optimal scenario will allow for the 
         algorithm to correctly determine the class labels for unseen instances. This requires 
         the learning algorithm to generalize from the training data to unseen situations in a 
         'reasonable' way (see inductive bias).
      """
kw_model = KeyBERT()
keywords = kw_model.extract_keywords(doc)
```

You can set `keyphrase_ngram_range` to set the length of the resulting keywords/keyphrases:

```python
>>> kw_model.extract_keywords(doc, keyphrase_ngram_range=(1, 1), stop_words=None)
[('learning', 0.4604),
 ('algorithm', 0.4556),
 ('training', 0.4487),
 ('class', 0.4086),
 ('mapping', 0.3700)]
```

To extract keyphrases, simply set `keyphrase_ngram_range` to (1, 2) or higher depending on the number 
of words you would like in the resulting keyphrases: 

```python
>>> kw_model.extract_keywords(doc, keyphrase_ngram_range=(1, 2), stop_words=None)
[('learning algorithm', 0.6978),
 ('machine learning', 0.6305),
 ('supervised learning', 0.5985),
 ('algorithm analyzes', 0.5860),
 ('learning function', 0.5850)]
``` 

We can highlight the keywords in the document by simply setting `hightlight`:

```python
keywords = kw_model.extract_keywords(doc, highlight=True)
``` 
  
**NOTE**: For a full overview of all possible transformer models see [sentence-transformer](https://www.sbert.net/docs/pretrained_models.html).
I would advise either `"all-MiniLM-L6-v2"` for English documents or `"paraphrase-multilingual-MiniLM-L12-v2"` 
for multi-lingual documents or any other language.  

###  Max Sum Similarity

To diversify the results, we take the 2 x top_n most similar words/phrases to the document.
Then, we take all top_n combinations from the 2 x top_n words and extract the combination 
that are the least similar to each other by cosine similarity.

```python
>>> kw_model.extract_keywords(doc, keyphrase_ngram_range=(3, 3), stop_words='english', 
                              use_maxsum=True, nr_candidates=20, top_n=5)
[('set training examples', 0.7504),
 ('generalize training data', 0.7727),
 ('requires learning algorithm', 0.5050),
 ('supervised learning algorithm', 0.3779),
 ('learning machine learning', 0.2891)]
``` 

###  Maximal Marginal Relevance

To diversify the results, we can use Maximal Margin Relevance (MMR) to create
keywords / keyphrases which is also based on cosine similarity. The results 
with **high diversity**:

```python
>>> kw_model.extract_keywords(doc, keyphrase_ngram_range=(3, 3), stop_words='english', 
                              use_mmr=True, diversity=0.7)
[('algorithm generalize training', 0.7727),
 ('labels unseen instances', 0.1649),
 ('new examples optimal', 0.4185),
 ('determine class labels', 0.4774),
 ('supervised learning algorithm', 0.7502)]
``` 

The results with **low diversity**:  

```python
>>> kw_model.extract_keywords(doc, keyphrase_ngram_range=(3, 3), stop_words='english', 
                              use_mmr=True, diversity=0.2)
[('algorithm generalize training', 0.7727),
 ('supervised learning algorithm', 0.7502),
 ('learning machine learning', 0.7577),
 ('learning algorithm analyzes', 0.7587),
 ('learning algorithm generalize', 0.7514)]
``` 

### Candidate Keywords/Keyphrases
In some cases, one might want to be using candidate keywords generated by other keyword algorithms or retrieved from a select list of possible keywords/keyphrases. In KeyBERT, you can easily use those candidate keywords to perform keyword extraction:

```python
import yake
from keybert import KeyBERT

doc = """
         Supervised learning is the machine learning task of learning a function that
         maps an input to an output based on example input-output pairs.[1] It infers a
         function from labeled training data consisting of a set of training examples.[2]
         In supervised learning, each example is a pair consisting of an input object
         (typically a vector) and a desired output value (also called the supervisory signal). 
         A supervised learning algorithm analyzes the training data and produces an inferred function, 
         which can be used for mapping new examples. An optimal scenario will allow for the 
         algorithm to correctly determine the class labels for unseen instances. This requires 
         the learning algorithm to generalize from the training data to unseen situations in a 
         'reasonable' way (see inductive bias).
      """

# Create candidates
kw_extractor = yake.KeywordExtractor(top=50)
candidates = kw_extractor.extract_keywords(doc)
candidates = [candidate[0] for candidate in candidates]

# KeyBERT init
kw_model = KeyBERT()
keywords = kw_model.extract_keywords(doc, candidates)
```

### Guided KeyBERT

Guided KeyBERT is similar to Guided Topic Modeling in that it tries to steer the training towards a set of seeded terms. When applying KeyBERT it automatically extracts the most related keywords to a specific document. However, there are times when stakeholders and users are looking for specific types of keywords. For example, when publishing an article on your website through contentful, you typically already know the global keywords related to the article. However, there might be a specific topic in the article that you would like to be extracted through the keywords. To achieve this, we simply give KeyBERT a set of related seeded keywords (it can also be a single one!) and search for keywords that are similar to both the document and the seeded keywords. 

Using this feature is as simple as defining a list of seeded keywords and passing them to KeyBERT:


```python
doc = """
         Supervised learning is the machine learning task of learning a function that
         maps an input to an output based on example input-output pairs.[1] It infers a
         function from labeled training data consisting of a set of training examples.[2]
         In supervised learning, each example is a pair consisting of an input object
         (typically a vector) and a desired output value (also called the supervisory signal). 
         A supervised learning algorithm analyzes the training data and produces an inferred function, 
         which can be used for mapping new examples. An optimal scenario will allow for the 
         algorithm to correctly determine the class labels for unseen instances. This requires 
         the learning algorithm to generalize from the training data to unseen situations in a 
         'reasonable' way (see inductive bias).
      """

kw_model = KeyBERT()
seed_keywords = ["information"]
keywords = kw_model.extract_keywords(doc, use_mmr=True, diversity=0.1, seed_keywords=seed_keywords)
```