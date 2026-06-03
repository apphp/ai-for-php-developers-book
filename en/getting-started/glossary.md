# Glossary

This chapter is a reference point. You can return to it whenever a term appears in code, a formula, or an algorithm description and you want to quickly refresh its meaning without diving into theory.

#### Accuracy (classification accuracy)

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

#### Algorithm

An algorithm is a finite sequence of steps that transforms input data into a result. In machine learning, an algorithm usually defines how we adjust the model parameters – for example, gradient descent, stochastic gradient descent, or k-nearest neighbors.

It is important to distinguish between a model and an algorithm: a model is the form of the relationship, while an algorithm is the method used to train or apply it.

#### **Affine Transformation**

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

#### **Attention Mechanism**

The attention mechanism is a way for a model to selectively "focus" on different parts of the input data depending on the task context.

The core idea of attention is that not all elements of the input are equally important. For each element, the model computes how strongly it is related to the current query and, based on this, weights its contribution.

In mathematical form, attention is built around dot products. For each element, three vectors are computed:

* **Query (Q)** – what we are currently looking for
* **Key (K)** – what each element represents
* **Value (V)** – what information it carries

The importance score is typically computed as the dot product $$Q ⋅ K$$, followed by normalization (most commonly softmax). The final result is a weighted sum of the vectors $$V$$.

Attention mechanisms are at the core of transformers and modern language models. They allow the model to capture dependencies between words regardless of their distance in the text, which fundamentally distinguishes them from classical sequential models.

Intuitively, attention can be seen as a dynamic focusing mechanism: the model continuously decides which parts of the input are important at each step. From a practical perspective, attention is a generalization of vector similarity, scaled up to the level of entire models.

#### Autoencoders

Autoencoders are neural network models that learn to compress data into a compact representation (code) and then reconstruct it.

They consist of two parts:

* encoder (compression)
* decoder (reconstruction)

They are used for dimensionality reduction, noise removal, and embedding training.

#### **Auto-scaling**

Auto-scaling is a mechanism for automatically adjusting a system’s computational resources based on workload.

In the context of machine learning, auto-scaling is applied to:

* model inference services
* data processing pipelines
* training and retraining systems

When the number of requests increases, the system automatically adds computational resources; when the load decreases, it releases them. This helps maintain stable response times and optimize infrastructure costs.

From a practical perspective, auto-scaling is not about the model itself, but about its lifecycle within a real system: even a good model becomes useless if it cannot handle the load.

#### **Backpropagation (error backpropagation)**

Backpropagation, or backprop, is an algorithm for computing the gradients of a loss function with respect to the parameters of a neural network by sequentially applying the chain rule from the model’s output back to its inputs.

The key idea of backprop is that the error computed at the network’s output is propagated backward through the layers, making it possible to determine the contribution of each parameter to the final error.

Mathematically, backprop is based on the chain rule of derivatives. Each layer receives the gradient from the next layer, multiplies it by its local derivative, and passes it further backward.

Backprop is not a standalone learning algorithm; rather, it is a mechanism for computing gradients. The parameters themselves are updated using gradient descent, SGD, or their variants.

In practical terms, backprop makes it possible to train deep, multi-layer models. Without it, computing gradients for neural networks would be computationally impractical.

Intuitively, backprop can be seen as a step-by-step analysis of the error: the model identifies which layer and which weights are responsible for an incorrect prediction and adjusts them accordingly.

#### **Bag-of-Words (BoW)**

Bag-of-Words (BoW) is a method for representing text as a numerical vector. Each element of the vector corresponds to a word in a vocabulary, and its value indicates how many times that word appears in the text.

In this representation, word order is completely ignored. For example, the phrases “PHP loves ML” and “ML loves PHP” will have identical representations.

BoW is simple, fast, and well-suited for initial experiments, but it does a poor job of capturing meaning and context.

#### **Base Rate Neglect**

Base rate neglect is a cognitive bias in which the prior probability of an event (the base rate) is ignored, and attention is focused only on new evidence or observed features.

In simpler terms, it is a situation where a person or a model fails to take into account how rare an event actually is.

**Example:**&#x20;

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

#### Batch

A batch is a group of samples that a model processes in a single training step.

Instead of using the entire dataset at once or updating parameters after each individual example, the model updates its parameters using small subsets of the data.

Intuitively, a batch is a "small portion of data for one training step".

#### Batch Gradient Descent (BGD)

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

#### Bayes' Formula

Bayes' formula allows us to update the probability of a hypothesis when new evidence becomes available:

$$
P(A\mid B)=\frac{P(B\mid A)\cdot P(A)}{P(B)}
$$

where:

* $$P(A)$$ – the prior probability
* $$P(B|A)$$ – the likelihood
* $$P(A|B)$$ – the posterior probability

Intuitively, Bayes' formula means: "update your initial belief based on new observations."<br>

#### Bayesian Updating

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

#### Bernoulli Naive Bayes

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

#### Bias

Bias is a model parameter that allows the output to be shifted independently of the input features. In linear regression, the bias corresponds to the term **b** in the formula:

$$
y=wx+b
$$

Intuitively, the bias represents the model's "baseline" prediction.

Without a bias term, the model would be forced to pass through the origin, which often makes it less flexible and reduces prediction accuracy.

In machine learning, the bias parameter is learned together with the model weights and helps the model better fit the data.<br>



\>>>>>>>>>

#### **Feature Engineering**

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

#### **Prior and Posterior Probability**

Prior probability is the probability of an event before taking new data into account.\
Posterior probability is the probability after incorporating observations.

The relationship between them is described by Bayes’ theorem:

$$
posterior ∝ likelihood × prior
$$

Intuitively, the prior is "what we believed beforehand", while the posterior is "what we believe after seeing the data."

Example:

If we know that 1% of emails are spam (the prior probability), and an email contains typical spam indicators, then after taking those indicators into account, the probability that the email is spam increases - that is the posterior probability.
