---
description: MSE, log loss – without formal hell.
---

# Error, loss functions, and why they are needed

### Error, Loss Functions, and Why We Need Them

Any machine learning model comes down to a simple idea: it tries to describe reality using some function. That means there will always be a mismatch between what actually exists and what the model predicts. We call this mismatch an error.

It is important to understand one key thing: the model does not know what is "good" or "bad". It does not understand the meaning of the task. During training, all it does is try to reduce a special number calculated from its errors. This number is called the loss function.

Formally, error is the deviation between the real value $$y$$ and the model prediction $$\hat{y}$$, while loss is a function that converts this deviation into a number convenient for optimization – a number the model tries to minimize during training.

Imagine a simple example. Suppose a model has to predict the price of an apartment.

```
Actual price: y = 200 000
Model prediction: ŷ = 180 000
```

The model made a mistake. The only question is how exactly we measure this error.

#### Error as Distance

Suppose we have a real value $$y$$ and a model prediction $$\hat{y}$$. The most natural thing to do is to look at the difference:

$$
error = y - \hat{y}
$$

But this value by itself is inconvenient for training a model. It can be both negative and positive. If we have many observations, positive and negative errors can cancel each other out.

For example:

```
+10
-10
```

The average error would be zero, even though the model is clearly making mistakes.

That is why we introduce a loss function – a function that transforms the error into a non-negative number suitable for optimization. This is the number the model tries to minimize during training.

Geometrically, it looks like this: we measure how far the prediction is from the real value on the number line.

Essentially, we want to convert the deviation into a number that behaves like a distance. Let us explain why. In regression tasks, it is convenient to interpret error as the distance between the real value and the prediction – that is, a number that is always non-negative and grows as the error increases.

However, not every loss function is a distance in the strict mathematical sense:

* In MSE – yes, it is the squared Euclidean distance between the prediction and the real value
* In log loss (logarithmic loss) – it is no longer a metric distance but a divergence

This immediately leads us to an important idea: error should be non-negative.

#### Squared Error as a Penalty for Missing the Target

The simplest way to get rid of the sign is to take the absolute value of the error.

$$
L = |y - ŷ|
$$

But the absolute value is not differentiable at point 0, which complicates optimization. That is why ML usually uses squared error instead:

$$
L = (y - \hat{y})^2
$$

Why?

First, squaring makes the function smooth and differentiable everywhere, which is critically important for gradient optimization and makes model training easier.

Second, squaring strongly amplifies large errors.

If the error increases by a factor of 2, the penalty increases by a factor of 4:

$$
(2e)^2 = 4e^2
$$

For example:

```
Error   →    Squared error
-----------------------------
 1                   1
 2                   4
 5                  25
10                 100
```

Because of this property, MSE is especially sensitive to large errors and outliers.

This is an important feature: we tell the model in advance that rare but large mistakes are worse than many small ones.

**Mean Squared Error (MSE)**

If we have many observations, we calculate the squared error for each of them and then average the results:

$$
\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
$$

From a geometric point of view, MSE is the average squared distance between real values and predictions – in other words, the squared vertical deviations of points from the model.

If we imagine the data as points on a plane and the model as a line or surface, MSE measures how far the points are from that surface.

**A Bit of Useful Math**

Why is MSE used so often? Because the minimum of MSE behaves very predictably.

If the model is linear:

$$
\hat{y} = wx + b
$$

then MSE as a function of parameters $$w$$ and $$b$$ is convex with respect to the model parameters.

This means:

* it has one global minimum
* the anti-gradient (the direction opposite to the gradient) points toward reducing the error function
* training is stable

This is one of the reasons why linear regression is such a fundamental and reliable tool.

**The Connection Between MSE and the Normal Distribution**

There is another important but often implicit fact. Minimizing MSE is equivalent to maximizing likelihood if we assume the errors are normally distributed:

$$
\varepsilon = (y - \hat{y}) \sim \mathcal{N}(0, \sigma^2)
$$

In this case, minimizing MSE is equivalent to maximizing likelihood.

In other words, MSE is not just a convenient formula. By using it, we implicitly assume that the noise $$\varepsilon$$ follows a normal distribution – the familiar "Gaussian bell curve".

**Why MSE Is Not Suitable for Classification**

So far we have discussed regression tasks, where the model predicts a numerical value. But there is another type of task – classification – where the model has to choose one of several possible answers.

Imagine a binary classification problem with two possible outputs: "yes" and "no". For example, spam or not spam.

Real value:

$$
y \in {0,1}
$$

Model prediction:

$$
\hat{p} \in [0, 1]
$$

If we use MSE, the difference between probabilities 0.99 and 0.51 does not appear very large, even though intuitively these predictions are of completely different quality when the correct answer is 1 (that is, spam).

In general, MSE does a poor job distinguishing confidence levels and does not match the probabilistic nature of the problem.

What matters is not only whether the model guessed correctly, but also how confident it is in its answer.

#### Log Loss as the Cost of Confidence

Log loss solves exactly this problem. The idea behind this function is very simple: it heavily penalizes the model when it is confidently wrong.

Suppose the correct answer is 1 (spam), meaning $$y = 1$$.

If the model predicts:

```
p = 0.9 → small error
p = 0.6 → larger error
p = 0.01 → enormous error
```

In other words, the more confidently the model is wrong, the stronger the penalty should be.

This idea is mathematically expressed by the log loss function.

For a single observation:

$$
LogLoss = - \left( y \log(\hat{p}) + (1 - y) \log(1 - \hat{p}) \right)
$$

If $$y = 1$$, only this term remains:

$$
-\log(\hat{p})
$$

Let us look at a few examples.

```
Probability  →    Loss
-----------------------
0.9               0.105
0.6               0.51
0.01              4.6
```

As we can see, being confidently wrong is extremely expensive.

In practice, the model makes predictions for many observations at once. Therefore, the total loss function is the average log loss across all observations:

$$
LogLoss = -\frac{1}{n} \sum_{i=1}^{n} \left(y_i \log(\hat{p}_i) + (1 - y_i) \log(1 - \hat{p}_i)\right)
$$

Consider a simple example with three observations.

```
y         p̂         loss
-------------------------
1        0.9        0.105
0        0.2        0.223
1        0.6        0.511
```

The average log loss will be:

$$
\frac{0.105 + 0.223 + 0.511}{3} \approx 0.28
$$

This is the value the model tries to minimize during training.

Now let us look at the graph of this function – it is very revealing.

* when $$\hat{p} \to 1$$, the error approaches zero
* when $$\hat{p} \to 0$$, the error approaches infinity

This is the mathematical expression of the idea:

> Being confidently wrong is almost a crime.

**The Geometric Meaning of Log Loss**

Log loss can be interpreted as a measure of divergence between the real distribution and the predicted probability distribution.

Formally, it is a special case of cross-entropy:

$$
H(p, q) = - \sum p \log(q)
$$

Where:

* $$p$$ – the true distribution
* $$q$$ – the model distribution

This makes log loss a natural choice for probabilistic models.

**Intuitive Comparison Between MSE and Log Loss**

MSE asks:

> How far off were we in value?

Log loss asks:

> How wrong were we in our confidence?

That is why the following practical rule works well in most real-world cases:

* regression → MSE
* classification → log loss

#### Final Thought

A loss function is the language we use to communicate with the model. Through it, we explain what we consider an error, which mistakes are especially bad, and which ones are acceptable.

The model knows nothing about money, spam, or the meaning of text. It only knows one thing: where to move in order to reduce the loss.

In the next chapter, we will see how minimizing loss turns into a concrete learning algorithm – through gradients and parameter updates.
