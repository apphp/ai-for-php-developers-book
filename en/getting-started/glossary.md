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

Here’s a clean English version adapted for your _AI with PHP_ book:

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

#### **Auto-scaling**

Auto-scaling is a mechanism for automatically adjusting a system’s computational resources based on workload.

In the context of machine learning, auto-scaling is applied to:

* model inference services
* data processing pipelines
* training and retraining systems

When the number of requests increases, the system automatically adds computational resources; when the load decreases, it releases them. This helps maintain stable response times and optimize infrastructure costs.

From a practical perspective, auto-scaling is not about the model itself, but about its lifecycle within a real system: even a good model becomes useless if it cannot handle the load.



\>>>



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
