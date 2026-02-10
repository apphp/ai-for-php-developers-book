# How this book is structured

In this book, I deliberately use professional jargon and terms from the fields of machine learning and software development. This is done for precision of wording and to align with established practice: many concepts do not have clear or concise equivalents in other languages. When key terms are first introduced, they are accompanied by brief explanations.

> Throughout the book, decimal notation with a dot as the decimal separator is used, as is customary in international scientific, technical, and programming practice.

### What This Book Covers

The text is written from an engineering perspective. There will be no academic mathematics for the sake of mathematics, and no abstract examples detached from real-world problems (well, almost none). We will gradually break down how machine learning works, starting from the most basic ideas and moving toward modern approaches such as embeddings and attention.

All examples are tied to PHP and typical backend scenarios. Even when we talk about neural networks, the focus will not be on formulas, but on understanding what is actually happening and how it can be used in practice.

In short, we will move through the following structure:

1. Core ML concepts without academic overload
2. Working with numbers, vectors, and matrices in PHP
3. Simple models – linear regression, classification
4. Embeddings and semantic search
5. Neural networks and attention at the level of understanding, not magic
6. Integration with pre-trained models
7. Architecture, limitations, and security

All examples in the book:

* are written in pure PHP or with minimal dependencies
* primarily use the **RubixML** library
* separate sections demonstrate working with **TransformersPHP**, **LLPhant**, and **NeuronAI**
* several examples are provided solely for demonstration purposes with **PHP-ML**
* focused on real-world backend tasks
* explained from an engineering perspective

### Who This Book Is Especially For

* PHP developers who want to understand AI, not just use it
* Team leads and architects making technical decisions
* Backend engineers working with text, search, and data
* Anyone tired of “magic” and wanting to understand how things work under the hood
