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

An **affine transformation** is a transformation of the form:

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

