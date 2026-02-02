# Notes

#### Additional Notes

**Ready-to-Use Environment**

You can install a ready-made Docker environment with examples:\
[https://github.com/apphp/ai-for-php-developers-examples](https://github.com/apphp/ai-for-php-developers-examples)

You can also run all examples from the book directly on my website:\
[https://aiwithphp.org/books/ai-for-php-developers/examples/](https://aiwithphp.org/books/ai-for-php-developers/examples/)

**Additional Notes on PHP-ML and Rubix ML**

1. PHP-ML:
   1. Provides a wide range of machine learning algorithms and preprocessing tools.
   2. Suitable for small datasets and simpler machine learning tasks.
   3. Easy to use thanks to a simple API.
2. Rubix ML:
   1. Offers a more comprehensive set of machine learning algorithms and tools.
   2. Better performance for larger datasets and more complex tasks.
   3. Provides more advanced features such as cross-validation and hyperparameter tuning.

When choosing between the two libraries, consider the complexity of your tasks, the size of your datasets, and the specific algorithms you need. In many projects, you can use both libraries to take advantage of their respective strengths.

#### **Troubleshooting**

If you encounter issues installing or using these libraries, follow these steps:

1. Make sure all required PHP extensions are installed. You may need to modify the Dockerfile to include additional extensions required by Rubix ML.
2. Check the library versions in the `composer.json` file and update them if necessary.
3. If you are working with large datasets, you may need to increase the memory limit in the PHP configuration.
4. Rubix ML may require additional system libraries. Add the following if you encounter issues:

For direct installation (run in the terminal)

```
apt-get install -y libopenblas-dev liblapacke-dev && docker-php-ext-install gd
```

For Docker (in the Dockerfile)

```dockerfile
RUN apt-get install -y libopenblas-dev liblapacke-dev && docker-php-ext-install gd
```
