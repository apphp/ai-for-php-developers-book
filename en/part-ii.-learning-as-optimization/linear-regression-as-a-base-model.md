---
description: Formula, geometric meaning, PHP implementation.
---

# Linear regression as a base model

### Linear Regression as a Fundamental Model

Linear regression is the point where it makes sense to begin talking about machine learning. Not because it is "simple", but because it already contains almost everything important.

All the key elements of machine learning already appear here:

* a model as a function
* parameters
* error
* optimization
* geometric meaning

If you understand linear regression, most other models will later feel like its extensions.

#### The Idea Behind the Model

Imagine we have data: inputs and correct answers. For example, an apartment area $$x$$ and its price $$y$$. We want to learn how to predict $$y$$ from the input $$x$$.

Linear regression assumes that the relationship can be approximated with a linear function:

$$
\hat{y} = w x + b
$$

{% hint style="info" %}
Linear regression was not invented in a single paper. Its foundations are connected to the least squares method, independently developed by [Carl Friedrich Gauss](https://ru.wikipedia.org/wiki/%D0%93%D0%B0%D1%83%D1%81%D1%81,_%D0%9A%D0%B0%D1%80%D0%BB_%D0%A4%D1%80%D0%B8%D0%B4%D1%80%D0%B8%D1%85) and [Adrien-Marie Legendre](https://en.wikipedia.org/wiki/Adrien-Marie_Legendre) at the beginning of the 19th century. Originally, the method was used in astronomy to process observational data.
{% endhint %}

This is just the standard equation of a line on a plane.

Where:

* $$x$$ – input feature
* $$w$$ – coefficient (weight)
* $$b$$ – bias (intercept)
* $$\hat{y}$$ – model prediction

If there are multiple features, the formula generalizes to:

$$
\hat{y} = w_1 x_1 + w_2 x_2 + \dots + w_n x_n + b
$$

Or in vector form, which is important for machine learning:

$$
\hat{y} = \mathbf{w} \cdot \mathbf{x} + b
$$

Here $$\mathbf{w}$$ are vectors, and the dot means a dot product.

In reality, the vector form is not a new model – it is simply a more convenient way to write the same sum.

Let us see how this transition happens.

**Expanded Form**

We already saw that the model can be written as:

$$
\hat{y}(x) = \hat{w}0 + \sum{j=1}^{p} x_j \hat{w}_j
$$

This means the model:

* multiplies each feature $$x_j$$ by its weight $$\hat{w}_j$$
* sums all the results
* adds the intercept $$\hat{w}_0$$

In other words, it is simply a sum of weighted features.

**How the Dot Product Appears**

Now let us rewrite the same formula using vectors.

We add a dummy feature equal to one:

$$
x =
\begin{pmatrix}
1 \\
x_1 \\
x_2 \\
\vdots \\
x_p
\end{pmatrix}
,\quad
\hat{w} =
\begin{pmatrix}
\hat{w}_0 \\
\hat{w}_1 \\
\hat{w}_2 \\
\vdots \\
\hat{w}_p
\end{pmatrix}
$$

Then their dot product:

$$
x^\top \hat{w}
$$

gives:

$$
x^\top \hat{w} =
1 \hat{w}_0 + x_1 \hat{w}_1 + \dots + x_p \hat{w}_p
$$

This is exactly the same sum as above.

The symbol ⊤ (read as "transpose") means that a vector or matrix is flipped – rows become columns and vice versa. This is needed to perform multiplication correctly:\
a row multiplied by a column gives a single number – exactly what we need for prediction.

That is why the entire linear model can be compactly written as a single dot product.

**Connection to the Previous Notation**

That is why these expressions:

$$
\hat{y} = \mathbf{w} \cdot \mathbf{x} + b
$$

and

$$
\hat{y}(x) = x^\top \hat{w}
$$

are equivalent.

In the second case, the bias $$b$$ is simply "embedded" into the weight vector through the added unit in $$\mathbf{x}$$.

**Why This Matters**

This notation is used because it is:

* more compact
* naturally generalized to matrices (for example, $$X\mathbf{w}$$ for the entire dataset)
* convenient for computation and optimization

In short:

$$
\hat{y}(x) = \hat{w}_0 + \sum x_j \hat{w}_j \quad \Longleftrightarrow \quad x^\top \hat{w}
$$

This is the same model written in two different ways:

* on the left – element by element
* on the right – in vector form

So, to summarize, the model answers the following question: what weights should we choose so that the linear combination of features matches the real data as closely as possible?

#### Error and the Loss Function

The model itself means nothing until we define what "good" and "bad" mean – for us, of course. That is why we introduce the concept of error.

For a single observation, the error looks like this:

$$
e = y - \hat{y}
$$

Strictly speaking, the quantity $$e$$ is called the residual, while the function we optimize (we will discuss it shortly) is called the loss function.

But optimizing the raw error is inconvenient – as we discussed in the previous chapter, positive and negative values cancel each other out. That is why classical linear regression almost always uses squared error:

$$
L = (y - \hat{y})^2
$$

And for the entire dataset – mean squared error (MSE):

$$
MSE = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
$$

The smaller the MSE, the more accurately the model predicts the data. This is exactly the quantity we minimize by selecting parameters $$w$$ and $$b$$.

#### Geometric Meaning

Geometry is the key to understanding linear regression.

**One Feature – A Line**

If we have one feature, the data are points on the plane $$(x, y)$$. The model is a line. Training linear regression means finding the line that passes "as close as possible" to these points. Closeness is measured by the sum of squared errors.

The vertical segments from the points to the line are the prediction errors.

**Multiple Features – A Plane and a Hyperplane**

If there are two features, the model becomes a plane. If there are more features – a hyperplane in multidimensional space.

The vector $$\mathbf{w}$$ defines the orientation of this plane (its tilt in space), while $$b$$ defines its shift.

The prediction $$\hat{y}$$ is the value of the linear function. Geometrically, it can be interpreted as the projection of the feature vector $$x$$ onto the direction of the weight vector $$\mathbf{w}$$, taking the bias $$b$$ into account.

From this perspective, linear regression is the problem of finding the direction in feature space that best explains the data.

#### How the Weights Are Found

There are two main approaches:

1. Analytical solution (using the normal equations)
2. Iterative optimization (gradient descent)

In applied ML, the second approach is more common because it scales better and logically matches how neural networks are trained.

#### Analytical Solution

The same error expression can conveniently be written in matrix form. If we collect all observations into a feature matrix $$x$$ and the targets into a vector $$y$$, the model can be written as:

$$
\hat{y} = X \mathbf{w}
$$

Then the loss function becomes:

$$
L(\mathbf{w}) = |X\mathbf{w} - y|^2
$$

This is the same sum of squared errors, just written more compactly.

**What We Minimize**

Let us expand the norm:

$$
L(\mathbf{w}) = (X\mathbf{w} - y)^\top (X\mathbf{w} - y)
$$

**Taking the Derivative**

Expanding the brackets gives:

$$
L(\mathbf{w}) = \mathbf{w}^\top X^\top X \mathbf{w} - 2 y^\top X \mathbf{w} + y^\top y
$$

Now differentiate with respect to $$\mathbf{w}$$:

* $$\mathbf{w}^\top X^\top X \mathbf{w} \rightarrow 2 X^\top X \mathbf{w}$$
* $$-2 y^\top X \mathbf{w} \rightarrow -2 X^\top y$$
* $$y^\top y \rightarrow 0$$

As a result:

$$
\nabla_{\mathbf{w}} L = 2X^\top X \mathbf{w} - 2X^\top y
$$

**Setting It Equal to Zero**

The minimum is reached when the gradient equals zero:

$$
2X^\top X \mathbf{w} - 2X^\top y = 0
$$

Dividing by 2 gives:

$$
X^\top X \mathbf{w} = X^\top y
$$

These are the normal equations.

**Solution**

If the matrix $$X^\top X$$ is invertible, the solution can be written as:

$$
\mathbf{w}^* = (X^\top X)^{-1} X^\top y
$$

**When the Solution Exists**

The matrix $$X^\top X$$ must be invertible.\
If the features are linearly dependent, it becomes singular, and:

* the inverse matrix does not exist
* the solution is either non-unique or undefined

**Intuitively**

* $$X^\top X$$ reflects how features are related to each other
* $$X^\top y$$ reflects how features are related to the targets
* the solution chooses weights that minimize the error

In short:

$$
\mathbf{w}^* = (X^\top X)^{-1} X^\top y
$$

is the analytical solution to the linear regression problem.

**Regularization: Ridge and Lasso**

In practice, the analytical solution does not always behave well: the model may overfit or become unstable.

One way to fix this is to add regularization.

**Ridge Regression (L2)**

In Ridge regression, a penalty on the weight magnitude is added to the loss function:

$$
L(\mathbf{w}) = |X\mathbf{w} - y|^2 + \lambda |\mathbf{w}|^2
$$

Here the second term is the L2 norm:

$$
|\mathbf{w}|^2 = \sum_j w_j^2
$$

This means that large weight values are penalized.

**What This Gives Us**

* the weights become smaller
* the model becomes more stable
* overfitting decreases

At the same time, the weights shrink smoothly but do not become zero.

**Ridge Solution**

In this case, the analytical solution changes slightly:

$$
\mathbf{w}^* = (X^\top X + \lambda I)^{-1} X^\top y
$$

The addition of $$\lambda I$$ stabilizes the matrix and guarantees that the solution exists (in other words, it prevents the matrix from becoming singular and the solution from "collapsing").

**Comparison with Lasso**

There is another type of regularization – L1:

$$
|\mathbf{w}|_1 = |w_1| + |w_2| + \dots
$$

It is used in Lasso regression.

The key difference:

* **L2 (Ridge)** – smooths the weights
* **L1 (Lasso)** – can drive weights to zero (performs feature selection)

In short:

* Ridge = L2 regularization
* Lasso = L1 regularization

#### Iterative Optimization (Gradient Descent)

The idea is simple: imagine the loss function as a surface. We stand at a random point and want to descend to the lowest point.

The gradient shows the direction of the steepest increase of the function. If we move in the opposite direction, the error decreases.

For linear regression, derivatives are easy to calculate. For clarity, let us write the derivatives for a single observation. For the full dataset, the gradients are averaged across all examples.

$$
\frac{\partial L}{\partial w} = -2 x (y - \hat{y})
$$

$$
\frac{\partial L}{\partial b} = -2 (y - \hat{y})
$$

These derivatives show how changes in the parameters affect the error.

The parameter update looks like this:

$$
w := w - \eta \cdot \frac{\partial L}{\partial w}
$$

$$
b := b - \eta \cdot \frac{\partial L}{\partial b}
$$

Where $$\eta$$ is the learning rate – the training step size.

At each step, the parameters shift slightly in the direction that reduces the error.

#### Analytical Solution vs Gradient Descent

A natural question arises: if an exact solution formula exists, why do we need gradient descent at all?

**Short Answer**

The formula

$$
\mathbf{w}^* = (X^\top X)^{-1} X^\top y
$$

is used because it:

* gives an exact solution
* does not require hyperparameter tuning
* works quickly on small problems

But it scales poorly.

**When the Formula Is a Good Choice**

The analytical solution is convenient if:

* the dataset is small (thousands, not millions)
* the number of features is relatively small
* you want an exact answer without tuning

For example:

* educational tasks
* statistical analysis
* small datasets

**Why It Does Not Scale**

The main problem is matrix inversion:

$$
(X^\top X)^{-1}
$$

This operation has computational complexity on the order of $$O(n^3)$$.

So:

* for small sizes – everything works quickly
* for large sizes – it becomes too expensive

**Why We Still Study It**

Despite this, the formula is important because it:

* gives the exact solution (the global minimum)
* helps understand the structure of the problem
* serves as a reference point for checking other methods

**How It Is Done in Practice**

* small datasets → analytical solution
* medium datasets → depends on the task
* large datasets → gradient descent / SGD

In short:

* the analytical solution is theoretically exact
* gradient descent is the practical tool for large-scale data

#### Why Linear Regression Is So Important

Linear regression may look simple, but it defines the fundamental pattern of model training:

* it establishes the basic "model" → "error" → "optimization" pipeline
* it reveals the geometric meaning of learning
* it teaches you to think in vectors and spaces
* it is directly connected to neural networks (a single neuron without activation is just a linear model)

In fact, every linear layer in a neural network is a generalization of linear regression. The only difference is the number of layers and the nonlinearities between them.

If you understand what is happening here, you already understand one of the most important parts of machine learning – regardless of programming language, framework, or trendy library.

In the following chapters, we will make the picture more complex and discuss why linearity is often not enough and how nonlinear models appear.
