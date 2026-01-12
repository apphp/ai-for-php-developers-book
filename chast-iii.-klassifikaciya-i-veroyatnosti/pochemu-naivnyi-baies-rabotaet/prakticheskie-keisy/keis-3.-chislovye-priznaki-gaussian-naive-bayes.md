---
description: Увидеть, что Байес не только про текст
---

# Кейс 3. Числовые признаки (Gaussian Naive Bayes)

## Числовые признаки (Gaussian Naive Bayes)

#### Сценарий

Классифицируем пользователя:

* active
* inactive

По:

* времени на сайте
* числу кликов

### Реализация: чистый PHP (идея)

```
function gaussian($x, $mean, $variance)
{
    return (1 / sqrt(2 * pi() * $variance)) *
           exp(-pow($x - $mean, 2) / (2 * $variance));
}
```

Здесь наивный Байес:

* оценивает среднее и дисперсию
* подставляет в формулу нормального распределения
* дальше все то же самое: произведение → лог-сумма

### RubixML

```
use Rubix\ML\Classifiers\GaussianNB;
use Rubix\ML\Datasets\Labeled;

$samples = [
    [120, 10],
    [130, 12],
    [20,  1],
    [30,  2],
];

$labels = ['active', 'active', 'inactive', 'inactive'];

$dataset = new Labeled($samples, $labels);

$model = new GaussianNB();
$model->train($dataset);

$prediction = $model->predict([[100, 9]]);
print_r($prediction);
```

