# Getting Started

### Introduction

This book grew out of a simple observation: PHP developers almost always find themselves on the opposite side of the barricades from machine learning. On one side is the world of web applications, APIs, databases, business logic, and real users. On the other is a world of articles about neural networks, overloaded with formulas, Python code, and a persistent feeling that this is "not for us". As a result, AI is either perceived as a black box or ignored altogether.

The goal of this book is to remove this artificial division.

A PHP developer already knows how to think in terms of models, even if they have never called it machine learning. We constantly deal with functions, parameters, constraints, errors, and approximations. We write code that takes input data and produces an output, expecting it to be "good enough". Machine learning is a continuation of this logic – not magic, and not a separate discipline reserved for a chosen few.

### Intuitively

There will be no attempt to throw the reader into linear algebra and statistics on the very first pages just for the sake of formulas. Every idea is first explained at the level of intuition: what we want to achieve, what problem we are solving, and why this approach makes sense at all. When a term appears, it is tied to concepts familiar to a programmer. A model is a function. Training is parameter selection. Prediction is a regular method call.

Intuition matters no less than formulas. Without it, mathematical derivations turn into a set of symbols that are hard to apply to real problems.

### Mathematically, but simple

At the same time, this book does not reduce machine learning to "press a button and get AI". If something is based on mathematics, that mathematics will be shown. Without excessive academic formalism, but also without hiding the hard parts.

We will talk about model error, what it means to "optimize", why the gradient works at all, where overfitting comes from, and why a model can perform excellently on tests and still be useless in a production environment. Formulas will appear only where it is impossible to preserve simplicity and clarity without them. Their purpose is not to intimidate, but to clarify what is happening under the hood. The complexity of these formulas will not go beyond high school algebra, basic calculus, and elementary statistics: sums, derivatives, and clear geometric interpretations, and so on.

### In practice

All examples and implementations are oriented toward PHP. Not as a "wrapper" around a Python script, but as a full-fledged environment for understanding and experimentation. We will implement simple models by hand: linear regression, classifiers, and elements of neural networks. Yes, not as efficient as in specialized libraries, but completely transparent.

This matters for two reasons. First, when you implement an algorithm yourself, it stops being abstract. Second, it helps you understand where the boundary lies: what makes sense to write in PHP, and where it is truly better to use external ML services or specialized tools.

### Who this book is for

This book is for PHP developers who:

* want to understand how AI works, not just call an API;
* are not afraid of code, but are tired of pseudo-scientific explanations;
* want to speak the same language as data scientists and ML engineers;
* want to apply machine learning in real products, not just in abstract textbook examples.

No deep mathematical background is required, but a willingness to think and ask questions is essential. Machine learning is not a set of recipes, but a way of thinking. And it is precisely this way of thinking that we will gradually build, step by step, relying on the experience of a PHP developer.

In the following chapters, we will start with the very foundation: what a model is in the mathematical sense, and why without this understanding it is impossible to talk about AI, neural networks, or "smart" applications at all.
