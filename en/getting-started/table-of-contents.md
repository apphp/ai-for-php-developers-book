# Table of Contents

{% hint style="info" %}
**Book development status**

The book is being written publicly and is in active development.

**Overall progress**

Readines&#x73;**:** 16% (31%)

🟩🟩🟩🟩🟨🟩🟩🟨🟨🟨🟨⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜⬜

Section statuses

🟩 - ready\
🟨 - at work\
⬜ - not started
{% endhint %}

### Getting Started

**Disclaimer**\
&#xNAN;_&#x41;bout the book's boundaries, assumptions and responsibilities._

**Introduction**\
_Motivation, realities and goals of the book._

* From understanding to mathematics and implementation
* Why a PHP developer needs AI

**The ML ecosystem in PHP**\
&#xNAN;_&#x41;n overview of the PHP ecosystem for machine learning and scientific computing._

* Environment setup for PHP

**About the book**

* License and copyright
* How to contribute
* Book change history
* What's next?

🟨 Glossary

**Resources and Literature**

#### Part I. The Mathematical Language of AI

**1.1 What a model is in the mathematical sense**\
&#xNAN;_&#x46;unction, parameters, error._

1.2 🟨 Vectors, dimensions, and feature spaces\
&#xNAN;_&#x57;hy data are points in space._

1.3 🟨 Distances and similarity\
&#xNAN;_&#x45;uclidean distance, dot product, cosine similarity._

&#x20;   1.3.1 Practical use cases

#### Part II. Learning as Optimization

2.1 🟨 Error, loss functions, and why they are needed\
&#xNAN;_&#x4D;SE, log loss – without formal hell._

&#x20;  2.1.1 Practical use cases

2.2 🟨 Linear regression as a base model\
&#xNAN;_&#x46;ormula, geometric meaning, PHP implementation._

&#x20;  2.2.1 Practical use cases

2.3 🟨 Gradient descent explained on fingers\
&#xNAN;_&#x57;hy the derivative is the direction of movement._

&#x20;  2.3.1 Experiments with gradient descent

#### Part III. Classification and Probabilities

3.1 ⬜ Probability as a degree of confidence\
&#xNAN;_&#x46;requencies, posterior probabilities._

&#x20;  3.1.1 Practical use cases

3.2 ⬜ Logistic regression\
&#xNAN;_&#x53;igmoid, decision boundary, classification case._

&#x20;  3.2.1 Practical use cases

3.3 ⬜ Why Naive Bayes works\
&#xNAN;_&#x43;onditional probabilities and independence._

&#x20;  3.3.1 Practical use cases

#### Part IV. Proximity and Data Structure

4.1 ⬜ The k-nearest neighbors algorithm and local decisions\
&#xNAN;_&#x47;eometric intuition, distance metrics._

&#x20;  4.1.1 Practical use cases

2.2 ⬜ Decision Trees and space partitioning\
&#xNAN;_&#x45;ntropy, information gain, explainability._

&#x20;  4.2.1 Practical use cases

#### Part V. Text as Mathematics

5.1 ⬜ Why words turn into numbers \
&#xNAN;_&#x57;ord spaces and features._

5.2 ⬜ Bag of Words and TF–IDF\
&#xNAN;_&#x46;ormulas, weight interpretation._

&#x20;  5.2.1 Practical use cases

5.3 ⬜ Embeddings as continuous spaces of meaning\
&#xNAN;_&#x47;eometry of meaning and semantic search._

5.4 ⬜ Transformers and context: from static vectors to understanding meaning \
&#xNAN;_&#x57;hy the word "key" has different vectors. Self-attention without formulas._

5.5 ⬜ Named Entity Recognition (NER) – extracting entities from text \
&#xNAN;_&#x53;equence labeling, BIO markup, practical case study in PHP._

5.6 ⬜ Practice: embeddings in PHP using transformers \
&#xNAN;_&#x49;nference instead of training. transformers-php as an engineering tool._

5.7 ⬜ RAG: Retrieval-Augmented Generation as an engineering system \
&#xNAN;_&#x53;earch → context → generation. Why an LLM "remembers" documents._

#### Part VI. Attention and Neural Networks

25. ⬜ Perceptron and fully connected network\
    &#xNAN;_&#x4C;inear combinations and nonlinearities._
26. ⬜ Backpropagation — why it works\
    &#xNAN;_&#x54;he chain rule without academic horror._
27. ⬜ Convolutional neural networks (CNN)\
    &#xNAN;_&#x46;ilters, local features, and why images are "understood" as matrices._
28. ⬜ Autoencoders\
    &#xNAN;_&#x44;ata compression, latent space and login recovery._
29. Recurrent neural networks (RNN)\
    &#xNAN;_&#x4D;odels that take into account the context over time._
30. ⬜ Attention as weighted summation

    _Q/K/V formulas + simplified implementation._

#### Part VII. LLMs and Modern AI

31. ⬜ Why LLMs are next-token prediction models\
    &#xNAN;_&#x50;robabilities, softmax, context._
    1. Hands-on: Building a Next-Token Model in PHP
    2. Practical use cases
32. ⬜ Why LLMs Behave "Intelligently": RLHF and Behavioral Learning\
    &#xNAN;_&#x46;rom Token Prediction to Useful Responses_
    1. Understanding RLHF in Practice
33. ⬜ Where LLMs fail mathematically

    _Hallucinations, distributions, bias._

#### Part VIII. Agent Systems and Orchestration

34. ⬜ Agent Systems and Multi-Step Reasoning in PHP\
    &#xNAN;_&#x4C;LM as a managed system: planning, tools and control._

#### Part IX. Production and Common Sense

35. ⬜ How to use AI in PHP projects\
    &#xNAN;_&#x41;rchitectures, integrations, orchestration._
36. ⬜ When AI is not needed and why it matters\
    &#xNAN;_&#x43;lear engineering boundaries._

#### Conclusion

37. ⬜ Where to go next\
    &#xNAN;_&#x57;hat to deepen: mathematics, systems, or practice._



{% hint style="success" %}
#### Support the book

If you find this material useful, you can support the development of the book.\
This helps continue the work and speeds up the release of new chapters.\
\
❤️ [Support the book](https://github.com/sponsors/apphp?o=esb)
{% endhint %}
