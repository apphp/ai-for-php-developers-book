---
description: >-
  An overview of the PHP ecosystem for machine learning and scientific
  computing.
---

# The ML Ecosystem in PHP

When people say that "there is no machine learning in PHP", they usually confuse two different things.

It is true that large neural networks are rarely trained from scratch in PHP (at least for now). But PHP has long and confidently lived in the world of _applying_ models: working with vectors, statistics, classification, embeddings, and numerical mathematics.

The ecosystem here is not noisy, but mature. It consists of four layers:

* classical ML libraries
* mathematical foundations
* integration tools for modern ML systems
* integration with external ML services

Let’s go through them one by one.

### Classical Machine Learning in PHP

Let’s start with libraries that implement machine learning algorithms directly in PHP, without calling external services.

#### **PHP-ML**

Repository: [https://github.com/jorgecasas/php-ml](https://github.com/jorgecasas/php-ml)

Status: <mark style="color:$warning;">not updated for a long time</mark>

PHP-ML is the starting point for understanding ML in PHP. It includes all the core algorithms from a classical ML course: k-NN, linear and logistic regression, Naive Bayes, SVM, decision trees, and k-means.

It’s important to understand the philosophy of PHP-ML. It does not try to compete with PyTorch or scikit-learn in terms of performance. Its goal is to give the developer a clear and honest ML tool directly inside PHP code. The code is easy to read, easy to debug, and easy to explain. For books and learning purposes, it is almost an ideal option.

A typical scenario: you have features from a database, you want to quickly train a model for classification or regression, save it, and use it at runtime without external services.

Let’s look at a simple but illustrative example. Here, we train a $$k$$-nearest neighbors (k-NN) classifier on a small set of points, each belonging to one of two classes — $$a$$ or $$b$$. After training, the model should determine which class a new point belongs to.

We define the training set as coordinates on a plane and their corresponding class labels:

```php
use Phpml\Classification\KNearestNeighbors;

$samples = [[1, 3], [1, 4], [2, 4], [3, 1], [4, 1], [4, 2]];
$labels = ['a', 'a', 'a', 'b', 'b', 'b'];

$classifier = new KNearestNeighbors();
$classifier->train($samples, $labels);

$prediction = $classifier->predict([3, 2]);

echo $prediction;
// Result: 'b'
```

{% hint style="info" %}
To test this code yourself, install the examples from the official [GitHub](https://github.com/apphp/ai-for-php-developers-examples) repository or use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/ml-ecosystem-in-php) to run it.
{% endhint %}

For the point $$[3, 2]$$, the algorithm returns class "$$b$$" because its nearest neighbors in the training set belong to that class. There is no "magic" here: k-NN simply looks at which points are closest and lets them "vote" based on their labels.

This transparency is exactly the algorithm’s main advantage. The code is easy to read, the model is easy to debug, and the mathematical meaning of what’s happening can be explained literally on your fingers — through distances between points and their local neighborhood.

But if PHP-ML is a "textbook", then the next library is already an "engineering tool".

#### **Rubix ML**

Website and repository: [https://github.com/RubixML](https://github.com/RubixML)

Status: <mark style="color:green;">**active**</mark>

Rubix ML is a full-fledged ML framework for PHP. It is oriented not toward demonstrating algorithms, but toward building reproducible ML pipelines: with data transformations, model serialization, strict interfaces, and a production-ready approach.

Rubix supports classification, regression, clustering, and treats datasets as first-class objects.

Let’s see what this looks like in practice.

Suppose we have data for binary classification. The code below also trains a $$k$$-nearest neighbors (k-NN) classifier, but this time on height and weight data with gender labels, and then predicts the label for a new person. For the parameters $$[172, 68]$$, the model returns "$$M$$" because the majority of the 3 nearest neighbors have this label.

```php
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Classifiers\KNearestNeighbors;

$samples = [
    [170, 65],
    [160, 50],
    [180, 80],
    [175, 70],
];

$labels = ['M', 'F', 'M', 'M'];

$dataset = new Labeled($samples, $labels);

$model = new KNearestNeighbors(3);
$model->train($dataset);

$testSamples = new Unlabeled([[172, 68]]);
$prediction = $model->predict($testSamples);

echo $prediction[0];
// Result: 'M'
```

{% hint style="info" %}
To test this code yourself, install the examples from the official [GitHub](https://github.com/apphp/ai-for-php-developers-examples) repository or use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/ml-ecosystem-in-php) to run it.
{% endhint %}

Notice an important point.

We are not passing "raw arrays" into the model — we are working with a `Dataset` object. This is a fundamentally different level of abstraction that immediately trains you to think like an ML engineer rather than the author of a script for a pet project.

Rubix can save models, apply transformers, scale features, and work with pipelines. In real PHP systems, this often becomes decisive, which is why Rubix is frequently chosen when a model is not an experiment but part of a long-lived service where versioning, reproducibility, and stability matter.

### Linear Algebra and Tensors as the Foundation of ML

Any ML code, even the most applied, ultimately boils down to operations on vectors and matrices. In PHP, there are several strong solutions for this.

#### **RubixML/Tensor**

Repository: [https://github.com/RubixML/Tensor](https://github.com/RubixML/Tensor)

Status: <mark style="color:green;">**active**</mark>

RubixML/Tensor is a low-level linear algebra library optimized specifically for machine learning tasks. It provides tensors, matrices, element-wise operations, transformations, and decompositions.

If Rubix ML is the "brain", then Tensor is the "muscle".

This library is especially important if you want to write ML code that not only works, but works stably and predictably in terms of memory usage and performance.

#### **MathPHP**

Repository: [https://github.com/markrogoyski/math-php](https://github.com/markrogoyski/math-php)

Status: <mark style="color:green;">**active**</mark>

MathPHP is a general-purpose mathematical library written in pure PHP. Linear algebra, statistics, probability, distributions, numerical methods.

In the context of machine learning, MathPHP is often used not directly for models, but as a computational foundation: distances, normalization, statistical estimates, hypothesis testing.

It’s a library that is ideal for explaining mathematics "plainly", without magic or hidden optimizations.

#### **NumPower**

Repository: [https://github.com/RubixML/numpower](https://github.com/RubixML/numpower)

Status: <mark style="color:green;">**active**</mark>

NumPower is a special case. It is a PHP extension for high-performance numerical computing inspired by NumPy. It uses AVX2 instructions on x86-64 and supports CUDA for GPU computations.

In effect, it answers the question: "Is it possible to do real numerical computing in PHP at the level of scientific computing"?

And the answer is yes — if you are willing to work with extensions and specific infrastructure.

NumPower is relevant where PHP is used not as a web layer, but as a computational engine.

#### **NumPHP and SciPhp**

NumPHP: [https://numphp.org/](https://numphp.org/)

Status: <mark style="color:$warning;">not updated for a long time</mark>

SciPhp: [https://sciphp.org/](https://sciphp.org/)

Status: <mark style="color:$warning;">not updated for a long time</mark>

These are libraries inspired by NumPy and scientific Python, but they have not been updated for a long time. Today, they are more of historical interest than a foundation for new projects.

Nevertheless, they are important for understanding that ideas of scientific computing in PHP appeared long before LLMs and the AI hype.

### Modern ML Integrations: Tokens, Embeddings, Data Pipelines

Modern machine learning rarely consists of just an "algorithm". There is always infrastructure around it: tokenization, data preparation, streaming processing.

#### **Tiktoken PHP**

Repository: [https://github.com/yethee/tiktoken-php](https://github.com/yethee/tiktoken-php)

Status: <mark style="color:green;">**active**</mark>

tiktoken-php is a PHP port of the OpenAI tokenizer. It is used for counting tokens, splitting text, and preparing data for LLMs.

If you work with GPT, Claude, or Gemini from PHP, this library becomes almost mandatory. It allows you to understand the real cost of requests, context length, and model behavior even before calling the API.

#### TransformersPHP

Repository: [https://github.com/CodeWithKyrian/transformers-php](https://github.com/CodeWithKyrian/transformers-php)

Status: <mark style="color:green;">**active**</mark>

Transformers PHP is one of the most interesting and illustrative projects in the modern PHP ML ecosystem. It is a library that allows you to use transformer models – BERT, RoBERTa, DistilBERT, and others – directly from PHP, without Python and without external APIs.

In essence, it is a PHP-oriented adaptation of the [Hugging Face Transformers](https://huggingface.co/docs/transformers/index) concepts, tailored to the PHP ecosystem and real-world application scenarios.

The key feature of the library is local inference. Models are loaded and executed on the PHP application side via [ONNX Runtime](https://onnxruntime.ai/). This enables important architectural advantages:

* no network calls to LLM APIs
* full control over data – critical for privacy
* predictable latency
* offline capability

Transformers PHP supports common NLP tasks:

* generating embeddings
* text classification
* semantic similarity
* feature extraction for downstream tasks

Conceptually, usage is simple: you load a pretrained model and apply it to text just as you would in Python – but entirely within PHP code. TransformersPHP provides a straightforward pipeline API for tasks such as sentiment analysis, text classification, semantic similarity, and more. In the example below, the model determines the sentiment of two phrases and returns a label and a score.

```php
use function Codewithkyrian\Transformers\Pipelines\pipeline;

// Выделить конвейер для анализа настроений
$classifier = pipeline('sentiment-analysis');

$out = $classifier(['I love transformers!']);
echo print_r($out, true);
// Array ( 
//   [label] => POSITIVE 
//   [score] => 0.99978870153427 
// )

$out = $classifier(['I hate transformers!']);
echo print_r($out, true);
// Array ( 
//   [label] => NEGATIVE 
//   [score] => 0.99863630533218 
// )
```

{% hint style="info" %}
To test this code yourself, install the examples from the official [GitHub](https://github.com/apphp/ai-for-php-developers-examples) repository or use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/ml-ecosystem-in-php) to run it.
{% endhint %}

It is important to understand the architectural role of Transformers PHP.

This library does not compete with large LLM services like GPT or Claude. It addresses a different – but very important – layer:

* fast embeddings
* local classification
* semantic search
* lightweight NLP without external dependencies

Within a PHP stack, this makes particular sense. PHP remains the central layer of business logic, while transformers become an embedded tool rather than a remote service.

Transformers PHP is a strong example of how modern ML is gradually becoming less "foreign" to PHP and turning into part of its native ecosystem – even if through careful engineering bridges such as ONNX.

#### **Rindow Math Matrix**

Repository: [https://github.com/rindow/rindow-math-matrix](https://github.com/rindow/rindow-math-matrix)

Status: <mark style="color:green;">**active**</mark>

Rindow Math Matrix is a linear algebra and matrix computation library focused on ML and numerical methods. It is often used together with other components of the Rindow ecosystem.

A good choice if you need a strict mathematical API and control over numerical operations.

#### **Flow PHP**

Repository: [https://github.com/flow-php/flow](https://github.com/flow-php/flow)

Status: <mark style="color:green;">**active**</mark>

Flow PHP is not an ML library in the strict sense, but a data processing framework. ETL, pipelines, transformations, validation, data streams.

In real ML systems, this layer often turns out to be the most complex. Data must be collected, cleaned, normalized, and only then fed into a model.

Flow PHP bridges the gap between "the data is somewhere" and "the model is already working".

### Integration with External ML Services

In practice, in most production scenarios, using ML in PHP means not training models, but performing inference via APIs.

#### **LLPhant**

Repository: [https://github.com/LLPhant/LLPhant](https://github.com/LLPhant/LLPhant?utm_source=chatgpt.com)

Status: <mark style="color:green;">**active**</mark>

LLPhant is a modern Generative AI framework for PHP that focuses not on "implementing an algorithm", but on "building an AI application".

If PHP-ML and Rubix operate at the level of models and mathematics, and TransformersPHP works at the level of local inference, LLPhant addresses the architectural layer around LLMs.

It is inspired by LangChain and LlamaIndex, but adapted to the PHP ecosystem.

LLPhant provides:

* a unified interface to multiple LLMs – OpenAI, Anthropic, Mistral, Ollama, and others
* embeddings support
* integration with vector stores – Redis, Doctrine, Qdrant, in-memory, and more
* RAG workflows – retrieval + generation
* chat memory and dialogue management
* tools for agents and tool calling

Architecturally, LLPhant is important because it allows you to treat an LLM not as "just an HTTP call", but as a system component. You work not simply with an API, but with abstractions:

* Chat
* Embeddings
* VectorStore
* Document
* Memory
* Pipeline

This moves LLM usage from the level of "a script that sends a request to OpenAI" to the level of reproducible architecture.

A simplest example:

```php
use LLPhant\Chat\OpenAIChat;
use LLPhant\Chat\Message;
use LLPhant\Chat\Enums\ChatRole;

$chat = new OpenAIChat();

$message = new Message();
$message->role = ChatRole::User;
$message->content = 'What is the capital of France?';

$response = $chat->generateText($message);
echo $response;

// Результат: 
// The capital of France is Paris.
```

{% hint style="info" %}
To test this code yourself, install the examples from the official [GitHub](https://github.com/apphp/ai-for-php-developers-examples) repository or use the [online demo](https://aiwithphp.org/books/ai-for-php-developers/examples/ml-ecosystem-in-php) to run it.
{% endhint %}

But the real power of LLPhant reveals itself in RAG scenarios:

1. You create embeddings for your documents.
2. Store them in a vector store.
3. When a user sends a query, you retrieve the relevant context.
4. Pass that context to the model.
5. Receive an answer grounded in your own data.

This is why LLPhant holds a special place in the PHP ML ecosystem. It is not a linear algebra library, not a classical machine learning toolkit, and not just a wrapper for local inference. It is a framework for building full AI systems on top of LLMs – with context management, data storage, and architectural structure.

In simple terms:

* PHP-ML – a textbook
* Rubix – an engineering tool
* TransformersPHP – local inference
* LLPhant – infrastructure for LLM applications

Projects like this show that the modern ML ecosystem in PHP is no longer just about "calling an API". It is becoming a full-fledged architectural layer around models.

#### **OpenAI, Anthropic, Gemini, and other LLMs**

Major LLMs have PHP SDKs or high-quality HTTP wrappers. Through them, PHP obtains:

* text embeddings
* classification
* generation
* summarization
* structured data extraction

From an architectural point of view, it looks like this: the model lives outside PHP, and PHP becomes the “business logic brain” that knows when and why to call ML.

This is exactly where PHP is especially strong: it integrates perfectly with queues, databases, caches, payments, and UI.

#### **ONNX Runtime and Model Inference**

ONNX deserves special mention. Through extensions or external services, PHP can run inference on models trained in Python and exported to the ONNX format.

This is a rare but important use case: the model is trained anywhere, but used in a PHP application without Python in production (via ONNX Runtime, extensions, or an external inference service).

#### **Computer Vision and Signal Processing**

In computer vision and signal processing tasks, PHP is not a leading language. However, basic tools for integrating such solutions do exist. OpenCV can be used via bindings, CLI calls, or external services, with PHP acting as the control and coordination layer.

In these scenarios, PHP rarely performs numerical computations directly. Its role is orchestration: launching and controlling CV pipelines, passing data between components, and integrating with queues, storage, and application business logic. This is a typical example of how PHP works effectively with ML components while remaining the central point of control rather than the computational core.

### How to Read This Ecosystem as a Whole

One conclusion is important to make before moving on.

PHP is not a language for image recognition competitions. It is a language that connects machine learning with real products. Its libraries are not optimized for accuracy benchmarks, but for: code clarity, integration, data control, and predictable behavior.

If you understand how a model works mathematically, PHP will give you enough tools to use it in production.

In this sense, the PHP + ML ecosystem is not poor — it is pragmatic and focused on applied tasks. PHP and machine learning are about PHP’s architectural role as a layer of integration, orchestration, and business logic around models:

* applying models
* working with embeddings and vectors
* classification and ranking
* orchestrating ML services
* connecting mathematics with business logic

That is why the PHP ecosystem looks not like a single monolith, but like a set of precise tools.

{% hint style="info" %}
For a broader overview of the ecosystem and current experiments, you can also refer to the curated Awesome PHP ML collection, which gathers libraries, tools, and projects related to machine learning in PHP:

[Awesome PHP ML](https://github.com/apphp/awesome-php-ml)
{% endhint %}
