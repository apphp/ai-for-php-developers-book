# 1.3.1 Practical use cases

In many machine learning tasks, objects are represented as feature vectors. To compare these objects, we need to formally define what "similar" or "different" means. This is exactly the role played by a distance or similarity measure.

**How to choose a distance measure**

Choosing a metric is essentially a way to formalize your understanding of how similar objects are.

* If geometric closeness and feature scale are important for the task, Euclidean distance is the natural choice. It works well in spaces where coordinates have a clear physical or numerical meaning.
* If the strength of feature interaction and their contribution to the result matter most, the dot product is used. It is the foundation of linear models and helps estimate how strongly two vectors "reinforce" each other.
* If meaning and direction are important, especially in text processing and embedding tasks, cosine similarity is used. It compares vectors independently of their length and works correctly with objects of different scales.

It is important to understand that a machine learning algorithm does not know what "similar" means by itself. It only operates on the numbers it receives as input. A distance measure translates your intuition and domain knowledge into the language of mathematics. This choice directly affects the behavior of the algorithm and the quality of the results.

On the following pages, we will look at real practical use cases where these distance measures are applied to solve specific tasks – with detailed explanations and PHP code examples:

* **Case 1: Comparing objects and users**\
  &#xNAN;_&#x55;sing Euclidean range._
* **Case 2: Estimating object relevance**\
  &#xNAN;_&#x55;sing the dot product._
* **Case 3: Comparing texts**\
  &#xNAN;_&#x55;sing cosine similarity._
