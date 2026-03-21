---
description: Function, parameters, error.
---

# What a model is in the mathematical sense

When we talk about a model in machine learning, it's helpful to immediately drop all associations with "artificial intelligence" and complex abstractions. In the mathematical sense, a model is a function. Nothing more, nothing less. It takes some input data and returns a result. The difference is that this function is not fixed – it has parameters that can be adjusted.

The easiest way to understand this is to look at it as a PHP developer. Any model is very similar to a regular class method or function that you write every day. There are input arguments, some computation inside, and a return value.

In the most general form, a model can be written like this:

$$
f(x) = ŷ
$$

Here, $$x$$ is the input data, and $$ŷ$$ is the model's prediction. The hat over y is intentional – it’s not the true value, just an attempt to estimate it.

#### **Function as the Foundation of a Model**

Suppose we want to predict the price of an apartment based on its area. In the simplest case, we can use a linear model:

$$
ŷ = w x + b
$$

This is already a complete model. It says: "The price ($$ŷ$$) is approximately equal to the area ($$x$$), multiplied by some coefficient ($$w$$), plus some offset ($$b$$)."

If we rewrite this in PHP, we get something very close to everyday code:

```php
class LinearModel {
    private float $w;
    private float $b;

    public function __construct(float $w, float $b) {
        $this->w = $w;
        $this->b = $b;
    }

    public function predict(float $x): float {
        return $this->w * $x + $this->b;
    }
}
```

Example:

```php
$model = new LinearModel(2.0, 0.0);
echo $model->predict(3.0);

// Result: 6
// Explanation: 2 * 3 + 0 = 6 
```

From a PHP perspective, this is just a regular class. No magic. But from a machine learning perspective – this is a model.

**Model Parameters**

The key point here is the parameters. In our example, these are ($$w$$) and ($$b$$). They are not input data, but they determine the behavior of the model. If you change ($$w$$) or ($$b$$), the function will produce different results.

It’s important to clearly distinguish between:

* input data ($$x$$) – what comes from the outside
* parameters ($$w, b$$) – what lives inside the model and is tuned
* output ($$ŷ$$) – the result of the model

In traditional programming, you usually hardcode parameters. In machine learning, it's the opposite – the structure of the function is fixed, and the parameters are learned automatically (during training).

#### **Error as a Measure of Quality**

Now comes the main question: how do we know that one set of parameters is better than another? For that, we introduce the concept of error, or a loss function.

Error is a function (commonly called a loss function) that compares the model’s prediction with the true value and returns a number that shows how wrong the model is. The smaller this number, the better the model.

For example, the simplest loss function is the difference between the prediction and the true value:

\$$\
ŷ - y\
\$$

```php
function error(float $yTrue, float $yPredicted): float {
    return $yPredicted - $yTrue;
}
```

Example:

```php
echo error(yTrue: 10.0, yPredicted: 7.0);

// Result: -3 
// Explanation: 7 - 10 = -3
```

In practice, the squared error (SE) is used more often, because it is always positive and penalizes larger mistakes more strongly:

\$$\
(ŷ - y) ^ 2\
\$$

```php
function squaredError(float $yTrue, float $yPredicted): float {
    return ($yPredicted - $yTrue) ** 2;
}
```

Example:

```php
echo squaredError(yTrue: 4.0, yPredicted: 6.0);

// Result: 4 
// Explanation: (6 - 4)² = 4
```

Notice an important detail. The error directly depends on the value of `$yPredicted`. Since `$yPredicted` is computed from the model parameters, the error ultimately becomes a function of the model parameters.

#### **Training as Error Minimization**

If we put everything together, we get a simple and very down-to-earth picture.

In general, we have:

* a model – a function with parameters
* data – pairs of inputs and correct answers
* error – a way to measure prediction quality

Training a model is the process of finding parameter values that minimize the total error on the data.

Even in PHP, this is easy to imagine as a loop. Primitive and inefficient, but conceptually correct:

```php
$dataset = [
    [1.0, 2.0],
    [2.0, 4.0],
];

$model = new LinearModel(w: 0.0, b: 0.0);

foreach ($dataset as [$x, $yTrue]) {
    $yPredicted = $model->predict($x);
    $loss = squaredError($yTrue, $yPredicted);

    // in real algorithms, parameters are updated here
    // so that the loss decreases
}

// Result:
x = 1, yTrue = 2, yPredicted = 0, loss = 4
x = 2, yTrue = 4, yPredicted = 0, loss = 16
```

Example:

```php
// Bad model (before training)
$model = new LinearModel(w: 0.0, b: 0.0);
// y = 0 * x
// Result:
// x = 1, yTrue = 2, yPredicted = 0, loss = 4
// x = 2, yTrue = 4, yPredicted = 0, loss = 16

// Improved model (after a few "training steps")
$model = new LinearModel(w: 0.8, b: 0.0);
// y = 0.8 * x
// Result:
// x = 1, yTrue = 2, yPredicted = 0.8, loss = 1.44
// x = 2, yTrue = 4, yPredicted = 2.4, loss = 5.76

// and so on – parameters are adjusted to reduce the loss
```

The specific way parameters are updated is a matter of optimization algorithms, which we’ll cover separately. The important idea here is different: no one "explains" rules to the model. We only tell it how wrong it is and let it gradually reduce that error.

#### **Why This Understanding Matters**

For a PHP developer, this perspective is especially important. It shows that machine learning does not contradict classical programming or replace it. On the contrary, it builds on the same basic ideas: functions, parameters, computations.

A model is not a black box. It’s a function.

Training is not magic. It’s optimization.

Error is not an abstraction. It’s just a function that returns a number.

If you understand this, the fear of machine learning should disappear. After that, it becomes engineering – which functions to use, which loss functions to choose, and how to efficiently find good parameters.

{% hint style="info" %}
To try this code yourself, use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/part-1/what-is-a-model) to run it.
{% endhint %}
