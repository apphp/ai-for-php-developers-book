# Glossary

This chapter is a reference point. You can return to it whenever a term appears in code, a formula, or an algorithm description and you want to quickly refresh its meaning without diving into theory.

### Accuracy (classification accuracy)

Accuracy is the fraction of correct predictions made by a model out of the total number of predictions. In simple terms, it answers the question: "In how many cases did the model guess correctly?"

Formally, accuracy is calculated as the ratio of correct predictions to the total number of examples.

Accuracy is convenient because it:

* is easy to compute
* is easy to interpret
* is intuitive

However, it has an important limitation: accuracy can be misleading on imbalanced datasets. For example, if 95% of the samples belong to one class, a model can achieve 95% accuracy simply by always predicting that class.

That is why accuracy is often used:

* as a baseline metric
* for an initial evaluation of a model
* together with other metrics (loss, precision, recall)

The intuitive meaning of accuracy is: "The percentage of cases where the model did not make a mistake." In machine learning, it is important to remember that high accuracy does not always mean a good model.

### Algorithm

An algorithm is a finite sequence of steps that transforms input data into a result. In machine learning, an algorithm usually defines how we adjust the model parameters – for example, gradient descent, stochastic gradient descent, or k-nearest neighbors.

It is important to distinguish between a model and an algorithm: a model is the form of the relationship, while an algorithm is the method used to train or apply it.

### Affine Transformation

An affine transformation is a transformation of the form:

$$
y=Wx+b
$$

where: $$W$$ is the weight matrix, $$x$$ is the input feature vector, and $$b$$ is the bias vector.

Geometrically, an affine transformation combines a linear transformation (rotations, scaling, stretching, compression, reflections) with a translation (shift). Unlike a purely linear transformation, it does not have to preserve the origin.

In machine learning, affine transformations form the foundation of most models:

* linear and logistic regression
* fully connected (dense) layers in neural networks
* feature and embedding transformations

In practice, almost any model first applies an affine transformation to the input data, and then maps it to a nonlinear representation using an activation function. That is why the expression $$Wx + b$$ can be considered a fundamental building block of modern machine learning.

Intuitively, an affine transformation determines how a model reweights input features and shifts the result into the appropriate region of the feature space.

### ANN Indexes (Approximate Nearest Neighbor)

ANN indexes are data structures designed for fast nearest-neighbor search in high-dimensional vector spaces.

Unlike exact search, ANN uses approximate algorithms to dramatically speed up retrieval while sacrificing only a small amount of accuracy.

They are widely used in:

* semantic search
* RAG (Retrieval-Augmented Generation) systems
* recommendation engines
* vector databases

Popular ANN algorithms include HNSW, IVF, and PQ.

Intuitively: "Find almost the nearest neighbors – but much faster."

### Attention Mechanism

The attention mechanism is a way for a model to selectively "focus" on different parts of the input data depending on the task context.

The core idea of attention is that not all elements of the input are equally important. For each element, the model computes how strongly it is related to the current query and, based on this, weights its contribution.

In mathematical form, attention is built around dot products. For each element, three vectors are computed:

* Query (Q) – what we are currently looking for
* Key (K) – what each element represents
* Value (V) – what information it carries

The importance score is typically computed as the dot product $$Q ⋅ K$$, followed by normalization (most commonly softmax). The final result is a weighted sum of the vectors $$V$$.

Attention mechanisms are at the core of transformers and modern language models. They allow the model to capture dependencies between words regardless of their distance in the text, which fundamentally distinguishes them from classical sequential models.

Intuitively, attention can be seen as a dynamic focusing mechanism: the model continuously decides which parts of the input are important at each step. From a practical perspective, attention is a generalization of vector similarity, scaled up to the level of entire models.

### Autoencoders

Autoencoders are neural network models that learn to compress data into a compact representation (code) and then reconstruct it.

They consist of two parts:

* encoder (compression)
* decoder (reconstruction)

They are used for dimensionality reduction, noise removal, and embedding training.

### Auto-scaling

Auto-scaling is a mechanism for automatically adjusting a system’s computational resources based on workload.

In the context of machine learning, auto-scaling is applied to:

* model inference services
* data processing pipelines
* training and retraining systems

When the number of requests increases, the system automatically adds computational resources; when the load decreases, it releases them. This helps maintain stable response times and optimize infrastructure costs.

From a practical perspective, auto-scaling is not about the model itself, but about its lifecycle within a real system: even a good model becomes useless if it cannot handle the load.

### Backpropagation (error backpropagation)

Backpropagation, or backprop, is an algorithm for computing the gradients of a loss function with respect to the parameters of a neural network by sequentially applying the chain rule from the model’s output back to its inputs.

The key idea of backprop is that the error computed at the network’s output is propagated backward through the layers, making it possible to determine the contribution of each parameter to the final error.

Mathematically, backprop is based on the chain rule of derivatives. Each layer receives the gradient from the next layer, multiplies it by its local derivative, and passes it further backward.

Backprop is not a standalone learning algorithm; rather, it is a mechanism for computing gradients. The parameters themselves are updated using gradient descent, SGD, or their variants.

In practical terms, backprop makes it possible to train deep, multi-layer models. Without it, computing gradients for neural networks would be computationally impractical.

Intuitively, backprop can be seen as a step-by-step analysis of the error: the model identifies which layer and which weights are responsible for an incorrect prediction and adjusts them accordingly.

### Bag-of-Words (BoW)

Bag-of-Words (BoW) is a method for representing text as a numerical vector. Each element of the vector corresponds to a word in a vocabulary, and its value indicates how many times that word appears in the text.

In this representation, word order is completely ignored. For example, the phrases “PHP loves ML” and “ML loves PHP” will have identical representations.

BoW is simple, fast, and well-suited for initial experiments, but it does a poor job of capturing meaning and context.

### Base Rate Neglect

Base rate neglect is a cognitive bias in which the prior probability of an event (the base rate) is ignored, and attention is focused only on new evidence or observed features.

In simpler terms, it is a situation where a person or a model fails to take into account how rare an event actually is.

Example:&#x20;

If a disease occurs in 1% of the population and a test has 95% accuracy, many people overestimate the probability of having the disease after a positive result. In practice, without considering the base rate, the true probability can be significantly lower than expected.

In machine learning, base rate neglect appears when:

* class imbalance is ignored
* prior probabilities are not taken into account
* models are evaluated using accuracy alone
* the data distribution is overlooked

This effect is directly related to:

* prior and posterior probabilities
* Bayes’ theorem
* probability calibration

Intuitively, base rate neglect is the error of thinking:

"The evidence looks convincing, so the event almost certainly occurred", even though the event itself may be extremely rare.

In practical applications, ignoring base rates can lead to overestimating risks, making incorrect decisions, and misinterpreting model results.

### Batch

A batch is a group of samples that a model processes in a single training step.

Instead of using the entire dataset at once or updating parameters after each individual example, the model updates its parameters using small subsets of the data.

Intuitively, a batch is a "small portion of data for one training step".

### Batch Gradient Descent (BGD)

Batch gradient descent is the classic version of gradient descent in which the gradient of the loss function is calculated using the entire training dataset before each model parameter update.

At each step, the algorithm uses all available data, so the gradient direction is exact for the current model. This makes the training trajectory smooth and stable.

Key characteristics of batch gradient descent:

* stable and predictable updates
* no gradient noise
* high computational cost on large datasets

Because it requires processing the entire dataset at every step, batch gradient descent scales poorly and is rarely used in its pure form for large-scale problems.

From a practical standpoint, batch gradient descent is well suited for:

* small datasets
* educational and analytical examples
* situations where reproducibility and update precision are important

In real-world systems, it is often replaced by mini-batch or stochastic gradient descent, which provide a better balance between training speed and stability.

### Bayes' Formula

Bayes' formula allows us to update the probability of a hypothesis when new evidence becomes available:

$$
P(A\mid B)=\frac{P(B\mid A)\cdot P(A)}{P(B)}
$$

where:

* $$P(A)$$ – the prior probability
* $$P(B|A)$$ – the likelihood
* $$P(A|B)$$ – the posterior probability

Intuitively, Bayes' formula means: "update your initial belief based on new observations."<br>

### Bayesian Updating

Bayesian updating is the process of recalculating the probability of a hypothesis when new data becomes available.

The idea is simple: we start with a prior probability – our initial assumption – then receive observations and update our confidence, producing a posterior probability.

Schematically:

new probability = old probability × effect of the data

Formally, this is expressed through Bayes' formula, but conceptually it comes down to gradually refining knowledge as more information accumulates.

Bayesian updating is used:

* in Naive Bayes
* in Bayesian models
* in risk assessment
* in online learning
* in systems that work with streaming data

Unlike the classical approach, where parameters are simply fitted, the Bayesian approach treats parameters as random variables and updates their probability distribution.

Intuitively, Bayesian updating follows this rule: "Do not start from scratch – adjust your confidence step by step."

In practical ML, this matters when data arrives gradually, the distribution changes, or the model needs to account for uncertainty explicitly.

### Bernoulli Naive Bayes

Bernoulli Naive Bayes is a variant of the Naive Bayes algorithm designed for binary features.

In this model, each feature can take only two values: present/absent, 1/0, or true/false.

Bernoulli Naive Bayes is most commonly used for text classification tasks, where a feature indicates whether a word appears in a document rather than how many times it appears.

The core idea is:

* the model estimates the probability of a feature being present for each class
* features are assumed to be independent
* the final class probability is computed as the product of individual probabilities

Bernoulli Naive Bayes works well when:

* the presence of a word is more important than its frequency
* the data is sparse
* a binary representation is used (bag-of-words without term frequencies)

The key difference from Multinomial Naive Bayes is:

* Bernoulli considers only whether a feature is present
* Multinomial takes feature frequency into account

Intuitively, Bernoulli Naive Bayes answers the question: "Which features are present, and how does their presence relate to a particular class?"

This model is simple, fast, and often serves as a strong baseline for text classification tasks.

### Bias

Bias is a model parameter that allows the output to be shifted independently of the input features. In linear regression, the bias corresponds to the term b in the formula:

$$
y=wx+b
$$

Intuitively, the bias represents the model's "baseline" prediction.

Without a bias term, the model would be forced to pass through the origin, which often makes it less flexible and reduces prediction accuracy.

In machine learning, the bias parameter is learned together with the model weights and helps the model better fit the data.

### Bias–Variance Tradeoff

The bias–variance tradeoff is a fundamental concept in machine learning that describes the balance between a model's simplicity and its ability to generalize to unseen data.

Bias is the error introduced by overly simplistic assumptions made by a model.

A model with high bias:

* is too simple
* fails to fit the training data well
* leads to underfitting

Variance measures how sensitive a model is to the specific training dataset.

A model with high variance:

* is too complex
* memorizes the training data
* generalizes poorly to new data
* leads to overfitting

The total prediction error of a model consists of:

* bias
* variance
* irreducible noise in the data

Increasing a model's complexity typically reduces bias but increases variance. Simplifying the model has the opposite effect.

In machine learning, the bias–variance tradeoff is managed through:

* model selection
* regularization (L1, L2)
* feature selection and the number of features
* training dataset size
* early stopping

Intuitively, the bias–variance tradeoff is about finding the right balance between "too simplistic" and "too perfectly fitted to the training data."

From a practical standpoint, understanding the bias–variance tradeoff is more important than knowing individual algorithms, because it explains why a model behaves the way it does.

### BM25 (Best Matching 25)

BM25 is a text retrieval algorithm that ranks documents by estimating how relevant they are to a search query based on keyword matches.

It takes into account:

* term frequency (TF) – how often a word appears in a document
* inverse document frequency (IDF) – how rare the word is across the entire collection
* document length

Unlike simple keyword search, BM25 provides a more accurate estimate of how well a document matches a query.

It is widely used in:

* search engines
* hybrid search
* RAG (Retrieval-Augmented Generation) systems alongside vector search

Intuitively: "How well does this document match the query based on the words it contains and their importance?"

### CART (Classification and Regression Trees)

CART is a decision tree algorithm used for both classification and regression tasks.

It recursively splits the data based on feature values, selecting the splits that most effectively reduce impurity (such as Gini impurity for classification or mean squared error (MSE) for regression).

Key characteristics:

* uses binary splits (each node is divided into two branches)
* supports both classification and regression
* is easy to interpret and visualize

Intuitively: "Ask a sequence of simple questions to progressively divide the data into more homogeneous groups."

### Chunking

Chunking is the process of splitting a large document into smaller pieces (chunks) that can be processed independently by a model.

Chunking is used when a document is too large to fit within a model's context window or when more efficient document retrieval is required.

It is commonly used in:

* RAG (Retrieval-Augmented Generation) systems
* semantic search
* vector databases
* processing long documents

The quality of chunking has a direct impact on the quality of both document retrieval and the model's responses.

Intuitively: "Break a large document into smaller, manageable pieces that are easier to search and process."

### Cold Start&#x20;

The cold start problem is a challenge in recommendation systems where the model lacks enough data to generate accurate recommendations.

This most commonly occurs in two situations:

* a new user, about whom nothing is known yet
* a new item (such as a product, article, or video) with no interaction history

The problem arises because traditional recommendation models rely on past behavior. When no historical data exists, the model has little or nothing to base its recommendations on.

Common ways to mitigate the cold start problem include:

* using content-based features (such as product descriptions, text, or categories)
* asking users for their initial preferences
* recommending popular or broadly appealing items
* combining multiple approaches using hybrid recommendation models

The cold start problem highlights an important principle: the quality of recommendations depends not only on the algorithm but also on the availability and structure of the data.

Intuitively: "You can't recommend based on someone's tastes if you don't know what they like yet."

### Concept Drift

Concept drift is a change in the statistical properties of data over time, where the relationship between the features and the target variable no longer remains the same.

Put simply, the model continues to operate, but reality has changed.

Examples of concept drift include:

* changes in user behavior
* seasonal effects
* the emergence of new types of data
* shifts in the market or external conditions

Concept drift is not a model error – it is a natural property of dynamic, real-world systems. Common approaches to handling it include performance monitoring, model retraining, sliding data windows, and online learning.

Intuitively, concept drift is the moment when the model has been left behind by reality.

### Content Deduplication

Content deduplication is the process of identifying and removing duplicate or highly similar content.

In machine learning and NLP, it is used to:

* clean training data
* reduce dataset size
* improve the quality of search and RAG systems

Duplicate content can lead to:

* model overfitting
* repetitive search results
* unnecessary storage and processing costs

Common techniques for detecting duplicates include:

* hashing
* comparing embedding vectors
* similarity metrics

Intuitively, content deduplication means removing repetitions so the model can learn and work with cleaner data.

### Context Window

A context window is the maximum number of tokens a model can consider at once when processing a request.

The context window includes:

* system instructions
* conversation history
* the user's prompt
* uploaded documents and any other relevant context

If the total amount of data exceeds the size of the context window, some information must be removed, summarized, or split into smaller chunks.

The size of the context window directly affects a model's ability to work with long documents and maintain coherent, long-running conversations.

Intuitively, the context window is the amount of working memory available to the model at any given moment.

### Contextual Embeddings

Contextual embeddings are vector representations of words or text that depend on the context in which they are used.

Unlike traditional embeddings, where each word is assigned a single fixed vector, contextual embeddings can produce different representations for the same word depending on its meaning in the surrounding text.

For example, the word "bank" has different embeddings in the following sentences:

* "The bank approved the loan."
* "They sat on the bank of the river."

Contextual embeddings are widely used in:

* transformer models
* NLP tasks
* semantic search
* RAG systems

Intuitively, the meaning of a word is determined not only by the word itself, but also by the context in which it appears.

### Cosine Similarity

**Cosine similarity** is a measure of similarity between two vectors based on the cosine of the angle between them:

$$
\cos(\theta)=\frac{\mathbf{A}\cdot\mathbf{B}}{|\mathbf{A}| |\mathbf{B}|}
$$

It measures how closely two vectors point in the same direction, regardless of their magnitude. This property makes it especially useful when working with text, embeddings, and TF-IDF vectors.

Cosine similarity is often preferred over Euclidean distance for text-based applications because what matters is not the absolute magnitude of the vectors, but the direction they point in—that is, their semantic similarity.

### CRF Layer (Conditional Random Field)

A CRF layer (Conditional Random Field) is a model layer that captures dependencies between consecutive predictions in a sequence.

Unlike standard classification, where each element is predicted independently, a CRF layer considers the entire sequence and learns which combinations of labels are most likely to occur together.

CRF layers have been widely used in NLP tasks such as:

* Named Entity Recognition (NER)
* Part-of-Speech (POS) tagging
* information extraction from text

For example, a CRF layer can learn that after the label "B-PER" (Beginning of a Person entity), it is much more likely to see "I-PER" (Inside the same Person entity) than an unrelated label.

Intuitively, a CRF layer allows the model to consider not only individual predictions, but also how those predictions fit together as a coherent sequence.

### Cross-Entropy

Cross-entropy is a loss function that measures the difference between the true probability distribution and the probability distribution predicted by a model.

Put simply, it measures how far the model's predicted probabilities are from reality.

If the model assigns a high probability to the correct class, the cross-entropy loss is low.

If the model is confidently wrong, the loss increases dramatically.

For this reason, cross-entropy penalizes confident mistakes much more heavily than uncertain ones.

Cross-entropy is widely used in:

* binary classification
* multiclass classification
* training neural networks
* NLP models

It is closely related to:

* log loss
* logistic regression
* Maximum Likelihood Estimation (MLE)
* information theory

Intuitively, cross-entropy answers the question: "How closely does the model's predicted probability distribution match the true distribution?"

In practical machine learning, minimizing cross-entropy means training a model that produces both accurate and well-calibrated class probabilities.

### Decision Boundary

A decision boundary is a boundary in the feature space that separates different classes.

Simply put, it is a line (in 2D), a surface (in 3D), or a hyperplane (in higher-dimensional spaces) that divides the feature space so that objects on one side are assigned to one class, while objects on the other side are assigned to another.

Different models produce different types of decision boundaries:

* Linear regression and logistic regression → a linear decision boundary
* k-NN → a complex, irregular ("jagged") boundary
* Decision trees → a stepwise, axis-aligned boundary
* Neural networks → can learn highly complex nonlinear boundaries

The complexity of a decision boundary is directly related to the bias–variance tradeoff:

* A boundary that is too simple → underfitting
* A boundary that is too complex → overfitting

Intuitively, a decision boundary answers the question: "Where is the line between the classes?"

In machine learning, it is important to understand that training a model is, in essence, the process of finding an appropriate decision boundary in the feature space.

### Dot Product

The **dot product** is an operation on two vectors that produces a single number. In coordinate form, it is defined as the sum of the products of the corresponding components:

$$
\mathbf{A} \cdot \mathbf{B} = \sum_i A_i B_i
$$

Geometrically, the dot product can be expressed in terms of the vectors' magnitudes and the angle between them:

$$
\mathbf{A} \cdot \mathbf{B} = |\mathbf{A}| |\mathbf{B}| \cos(\theta)
$$

This formula directly leads to cosine similarity, which is simply the normalized dot product.

In machine learning, the dot product appears everywhere. A linear model computes its prediction as the dot product of the feature vector and the weight vector, plus a bias term. Similarity search, embeddings, TF-IDF, and Bag-of-Words representations also rely heavily on dot products between vectors.

Intuitively, the dot product answers the question: "How closely do two vectors point in the same direction?" If the result is large and positive, the vectors are similar; if it is close to zero, they are nearly unrelated; if it is negative, they point in opposite directions.

From a practical perspective, the dot product is one of the fundamental operations underlying both classical and modern machine learning algorithms.

\>>>>>>>>>

### Feature Engineering

Feature engineering is the process of transforming raw data into features that a model can effectively work with.

A model does not inherently understand text, events, users, or real-world numbers. It operates only on feature vectors. Feature engineering determines what those vectors will look like.

Feature engineering includes:

* normalization and scaling of numerical data
* encoding of categorical features (one-hot encoding, target encoding)
* text processing (stop words removal, stemming, lemmatization, TF-IDF, embeddings)
* generation of new features from existing ones
* aggregations, time windows, counters, and flags

The quality of feature engineering often has a greater impact on performance than the choice of a specific algorithm. A simple model with well-designed features can outperform a complex model with poor ones.

Intuitively, feature engineering is the process of translating reality into a language that the model can understand.

From a practical perspective, feature engineering is a key competency in applied machine learning, especially when working with limited data and classical algorithms.

### One-hot encoding

One-hot encoding is a method for converting categorical features into a numerical representation using binary vectors.

Each category corresponds to a separate dimension of the vector. For a given object, exactly one dimension is set to 1, while all the others are set to 0.

For example, the feature "color" with the values {red, green, blue} can be encoded as follows:

* red → \[1, 0, 0]
* green → \[0, 1, 0]
* blue → \[0, 0, 1]

One-hot encoding is widely used in linear models, logistic regression, and neural networks when the categories do not have a natural ordering.

The main drawback of one-hot encoding is the increase in the dimensionality of the feature space when the number of categories is large. In text processing, Bag-of-Words and TF-IDF are essentially special cases of one-hot representations weighted by term frequency.

In more advanced models and when dealing with a large number of categories, one-hot encoding is often replaced by embeddings, which provide a compact representation of categorical data while capturing latent relationships between categories.

### Prior and Posterior Probability

Prior probability is the probability of an event before taking new data into account.\
Posterior probability is the probability after incorporating observations.

The relationship between them is described by Bayes’ theorem:

$$
posterior ∝ likelihood × prior
$$

Intuitively, the prior is "what we believed beforehand", while the posterior is "what we believe after seeing the data."

Example:

If we know that 1% of emails are spam (the prior probability), and an email contains typical spam indicators, then after taking those indicators into account, the probability that the email is spam increases - that is the posterior probability.

### Vector

A vector is an ordered collection of numbers that represents an object. In machine learning, almost everything is a vector: a piece of text, a user, an event, or an image.

Working with vectors is one of the fundamental building blocks of machine learning.

