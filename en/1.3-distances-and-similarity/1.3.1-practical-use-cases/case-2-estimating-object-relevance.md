# Case 2: Estimating object relevance

The dot product is used in problems where neither the distance between objects nor just their direction in feature space is what matters. Instead, what matters is how much each feature contributes to the final result. This idea is at the core of linear models, scoring systems, and most ranking algorithms.

#### **Problem Statement**

Suppose we need to evaluate the relevance of an object. We have a collection of objects (for example, users, products, or documents), and each object is described by a set of numerical features:

* usage frequency
* activity level
* recency of interaction

Each feature contributes differently to the final score. We encode this knowledge as a weight vector.

An object is represented by the feature vector

$$
\mathbf{x} = (x_1, x_2, \dots, x_n)
$$

The model is represented by the weight vector

$$
\mathbf{w} = (w_1, w_2, \dots, w_n)
$$

#### **Why the Dot Product?**

The dot product computes the weighted sum of the features:

\$$\
\mathbf{x} \cdot \mathbf{w} = \sum\_i x\_i w\_i\
\$$

In this scenario:

* the **magnitude of the features** matters, not just their relative proportions
* the scale of the values is meaningful
* the contribution of each feature is explicitly controlled by its weight

This is why neither Euclidean distance nor cosine similarity is appropriate here.

**Option 1. Pure PHP Implementation**

**Dot product**

```php
function dotProduct(array $a, array $b): float {
    $n = count($a);

    if ($n !== count($b)) {
        throw new InvalidArgumentException('Vectors must have the same length');
    }

    $sum = 0.0;

    for ($i = 0; $i < $n; $i++) {
        $sum += $a[$i] * $b[$i];
    }

    return $sum;
}
```

**Example: object scoring**

```php
$features = [10, 5, 2];   // activity, sessions, purchases
$weights  = [0.3, 0.5, 1.5];

$score = dotProduct($features, $weights);

echo $score;

// Result: 8.5
// Explanation: 10 * 0.3 + 5 * 0.5 + 2 * 1.5 = 8.5
```

The higher the value, the higher the object's relevance score.

**Interpreting the result**

* the first feature contributes 3.0
* the second contributes 2.5
* the third contributes 3.0

The result is easy to interpret and explain, which is especially valuable in real-world applications.

**Option 2. Implementation Using Rubix ML**

In Rubix ML, the dot product is used internally by linear models. Let's look at an example using linear regression with weights learned from a small training dataset.

**Example: linear regression-based scoring**

```php
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Regressors\Ridge;

$samples = [
    [10, 5, 2],   // object A
    [4, 1, 0],    // object B
    [20, 8, 5],   // object C
];

// Dummy target values (for demonstration)
$labels = [8, 2, 15];

$dataset = new Labeled($samples, $labels);

$model = new Ridge(1.0);
$model->train($dataset);

// New object
$newSample = [[9, 6, 4]];

$prediction = $model->predict(new Unlabeled($newSample));
print_r($prediction);

// Result:
// Array (
//     [0] => 8.175
// )
```

**What happens internally**

Ridge is a linear regression model with [L2 regularization](https://chatgpt.com/ai-for-php-developers/introduction/glossary.md#l2-regularization) (that is, Ridge = Linear Regression + L2):

\$$\
\hat y = w\_1 x\_1+w\_2 x\_2+w\_3 x\_3 + b\
\$$

where the weights are obtained by solving:

\$$\
\mathbf{w} = \left( \mathbf{X}^\top \mathbf{X} + \alpha \mathbf{I} \right)^{-1} \mathbf{X}^\top \mathbf{y}\
\$$

**Relationship to Linear Models**

The dot product is the core operation behind linear models:

* linear regression
* logistic regression
* linear classifiers
* neural networks (at the level of individual neurons)

In fact, every neuron computes the dot product of its inputs and weights.

**Summary**

\{% hint style="warning" %\}\
Important: The dot product is used whenever the contribution of each feature to the final result matters. It provides a direct way to model feature influence and forms the foundation of linear models and scoring systems.\
\{% endhint %\}

This idea becomes even more important when we move on to trainable models, where the weights (\$$\mathbf{w}\$$) are learned automatically from data.

\{% hint style="info" %\}\
To experiment with this code yourself, use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/part-1/distances-and-similarity).\
\{% endhint %\}
