---
description: Word spaces and features.
---

# Why words turn into numbers

In previous chapters, we discussed how the world looks like a collection of numbers to an algorithm. Temperature, price, age, distance – all of these can easily be represented as numbers, and we usually have no problem accepting that. But as soon as we start working with text, our intuition begins to resist the idea.

Words carry meaning. Phrases provide context. Text contains culture, irony, hints, and emotions. How can we possibly turn all of this into a set of numbers in a way that allows a machine to "understand" something?

The answer is both simple and a little disappointing: a machine does not understand anything. It can perform calculations, work with numbers, and reproduce statistical patterns. Here, "understanding" means human, meaningful understanding, not the formal processing of numbers. Our task is to design a numerical representation of text that makes mathematical calculations correlate with meaning.

This chapter explains why we need numbers to work with text, how a word space emerges, and what features mean when we are dealing not with height and weight, but with sentences and documents.

#### Text as an Object for Computation

Most machine learning algorithms, including those we have encountered in previous chapters, take a vector of numbers as input:

$$
x = (x_1, x_2, \dots, x_n)
$$

Linear regression, logistic regression, k-NN, and decision trees all operate in numerical space. They have no operations such as "if the word is sad" or "if the phrase sounds aggressive". They only perform comparisons, additions, multiplications, and distance calculations.

That is why the first and unavoidable step when working with text is to map it into a numerical space. Mathematically, this looks like this:

$$
f: text → \mathbb{R}^n
$$

On the left, we have a string of characters. On the right, we have a point in an $$n$$-dimensional space.

We need to accept one uncomfortable fact from the start: there is no single "correct" way to map text to numbers. There are only representations that are more or less useful for specific tasks.

#### From Words to Coordinates

Let's start with an intuitive example. Imagine a very simple world containing only three words:

"cat", "dog", and "car".

We want to represent them as points. The most straightforward approach is simply to assign each word a number:

```
cat → 1
dog → 2
car → 3
```

Technically, these are numbers, but in practice, this representation is useless. Why?

Because the distance between 1 and 2 is smaller than the distance between 1 and 3. But that does not mean that "cat" is somehow closer to "dog" than to "car". The numbers imply an ordering that does not exist in the words themselves.

The model may start using numerical patterns that have no meaningful interpretation for us. In other words, it may find patterns where none actually exist. This is especially problematic for models that rely on distances and linear combinations of features.

This leads to a key idea: numerical values should reflect meaningful properties of objects rather than serve merely as formal identifiers.

#### Features Instead of Words

When we talk about a "feature", we mean a measurable characteristic of an object. For text, the object can be a word, a sentence, or a document. Features are answers to questions like these, which do not necessarily need to be directly interpretable by humans:

* Does this word appear?
* How many times does it appear?
* Does it appear near other words?
* How frequently does it appear in this document compared to other documents?
* And so on.

Each question gives us a number. A collection of these numbers turns the text into a vector.

If our vocabulary contains $$N$$ words, the simplest approach is to create one feature for each word.

A document can then be represented as a vector of length $$V$$:

$$
x = (c_1, c_2, \dots, c_n)
$$

where $$c_i$$ is the number of occurrences of the $$i$$-th word in the document.

This is now a genuine numerical representation that allows us to calculate distances, construct hyperplanes, and optimize loss functions.

#### Word Space

When each feature represents a word, we get a word space. Geometrically, this means the following:

* Each word corresponds to an axis, as in the [bag-of-words](../getting-started/glossary.md#bag-of-words-bow) model.
* Each document is a point.
* The similarity between documents is determined by their vocabulary.

If two documents use similar words with similar frequencies, their vectors will be close to each other.

If the vocabulary contains 10,000 words, the space has 10,000 dimensions. This may sound intimidating, but it is perfectly normal from a mathematical perspective. All the calculations remain the same; the vectors are simply longer.

Here is an interesting detail: most coordinates in these vectors are zero. A typical document uses only a small fraction of the entire vocabulary.

This brings us to sparse vectors and sparse representations – data structures that store only non-zero values. Without them, working with text would be computationally inefficient and often impractical.

#### A Little Math: Distances Between Texts

Once text has been converted into a vector, we can measure the distance between documents, as we discussed in previous chapters.

For example, the Euclidean distance:

$$
|x - y| = \sqrt{\sum_i (x_i - y_i)^2}
$$

Or cosine similarity:

$$
\text{cosine_sim}(x, y) = \frac{x \cdot y}{|x| \ |y|}
$$

Cosine similarity is particularly important for text because it compares the direction of vectors rather than their lengths. This means that two documents can be considered similar even if one is ten times longer than the other, provided that they use the same words in similar proportions and have similar relative word frequencies.

#### A Detailed Example: Two Texts → Two Vectors

Let's look at a very simple, deliberately toy example. Suppose we have two short texts:

```
Text A: "cat sleeps on the sofa"
Text B: "dog sleeps on the sofa"
```

From a human perspective, the texts are almost identical. Only one word is different. Let's see how an algorithm represents them.

Suppose our vocabulary contains just five words:

$$
{ cat, dog, sleeps, on, sofa }
$$

Now let's represent each text as a word-frequency vector using the bag-of-words approach, which we will examine in more detail in the next chapter.

For text A:

$$
x_A = (1, 0, 1, 1, 1)
$$

For text B:

$$
x_B = (0, 1, 1, 1, 1)
$$

Geometrically, these are two points in a five-dimensional space. They are very close because they differ only in the coordinates corresponding to one word.

If we calculate the Euclidean distance between them, it will be small, while their cosine similarity will be high. The algorithm "sees" that the texts are almost identical, even though it does not know what a cat or a dog is.

Now consider an important detail. Suppose we have a third text:

```
Text C: "car broke down on the road"
```

If we expand the vocabulary, its vector will differ in almost every coordinate. The distance between text C and texts A and B will be significantly greater.

<details>

<summary><strong>Example with an Expanded Vocabulary</strong></summary>

Our new vocabulary:  $${\{  cat, dog, sleeps, on, sofa, car, broke, down, road \}}$$

Word-frequency vector for text A:

$$x_A = (1, 0, 1, 1, 1, 0, 0, 0, 0)$$

Word-frequency vector for text B:

$$x_B = (0, 1, 1, 1, 1, 0, 0, 0, 0)$$

Word-frequency vector for text C:

$$x_C = (0, 0, 0, 1, 0, 1, 1, 1, 1)$$

</details>

This is how a statistical approximation of what humans perceive as semantic similarity emerges from pure geometry.

#### Why Dimensionality Is Not a Problem

When we say that a word space can have 10,000 or even 100,000 dimensions, it often sounds intimidating. It seems that the mathematics must become incredibly complicated.

But it is important to distinguish between two different sources of complexity: mathematical complexity and computational complexity.

From a mathematical perspective, almost nothing changes, at least at the level of formulas, rather than their statistical properties. A vector remains a vector, a distance remains a distance, and a dot product remains a dot product:

$$
x \cdot y = \sum_i x_i y_i
$$

The formulas do not become more complicated as dimensionality increases. However, the interpretation of distances and the behavior of the data change considerably. The calculations themselves simply involve summing more terms.

The complexity arises from the data and the resources required to process it: memory, computation time, and the storage of sparse structures. That is why machine learning places so much emphasis on optimization and data structures, rather than on making the formulas more complicated.

There is another important point: high dimensionality is not inherently a mistake. It reflects the complexity of language, although it also introduces its own statistical and computational challenges. The richer the language, the more axes we need to represent it.

#### A Mini Counterexample: Where Word Space Breaks Down

Consider these two sentences:

```
"This is a good movie"
"This is not a good movie"
```

For a human, the difference is fundamental. For a bag-of-words model, it is almost negligible.

If we ignore word order and negation, the two sentences will have almost identical vectors. The word "not" adds just one coordinate, but the overall vectors will still be very similar.

The algorithm is not "making a mistake". It is working exactly as expected within the space we have defined for it.

This counterexample is useful not as a criticism of the approach, but as a reminder of the engineer's responsibility. We decide which features exist and what geometry the text space will have.

#### Why This Works, Even Though It Looks Rather Naive

At first glance, the "one word = one feature" approach seems crude. It ignores word order, grammar, irony, sarcasm, and much of what we consider meaning.

Yet in practice, it often works surprisingly well. This is especially true for classification, search, and topic analysis, where word statistics matter more than the precise semantics of individual sentences.

The reason lies in statistics. In real-world text, words are not distributed randomly. Topics, emotions, and intentions are reflected in word frequencies and combinations.

An algorithm does not understand what "sadness" means, but it can detect that words such as "bad", "tired", and "meaningless" often appear together and correlate with a particular label. That is why even simple models can produce useful results.

Meaning does not emerge inside the model alone. It also depends on the data and how we define the task.

#### Limitations of Word Space

It is important to understand where this approach breaks down.

It cannot distinguish between:

* "The cat bit the dog" and "The dog bit the cat".
* "This is good" and "This is not good".
* Words that have similar meanings but different spellings.

These limitations motivate the development of more sophisticated representations: N-grams, TF–IDF, embeddings, and contextual models.

But the foundation remains the same: text is converted into numbers, the numbers form a space, and mathematics takes it from there.

#### The Key Takeaway

For an algorithm, text is not words – it is coordinates.

We are not "teaching a machine to understand language". We are designing a feature space in which statistical methods can produce meaningful results.

In the following chapters, we will make this space more sophisticated, less naive, and more expressive. But the idea of mapping text into $$\mathbb{R}^n$$ will remain with us throughout the entire book.
