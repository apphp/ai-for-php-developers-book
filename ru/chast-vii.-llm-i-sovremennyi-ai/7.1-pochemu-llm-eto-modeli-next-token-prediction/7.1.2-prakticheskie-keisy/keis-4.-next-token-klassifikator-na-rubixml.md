# Кейс 4. Next-token классификатор на RubixML

Теперь сделаем более “настоящий” ML-пример.

Используем [Rubix ML](chatgpt://generic-entity?number=1).

#### Задача

Предсказать следующий класс токена:

Например:

* normal
* phishing
* urgent

Мы превращаем контекст в числовые признаки.

#### Пример кода

```
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Classifiers\MultilayerPerceptron;

$samples = [
    [0.1, 0.8, 0.3],
    [0.9, 0.2, 0.4],
    [0.05, 0.9, 0.2],
];

$labels = ['normal', 'phishing', 'normal'];

$dataset = new Labeled($samples, $labels);

$estimator = new MultilayerPerceptron([5, 5]);
$estimator->train($dataset);

$prediction = $estimator->predict([[0.2, 0.7, 0.3]]);

print_r($prediction);
```

### Связь с LLM

Это всё тот же принцип:

P(class \mid features)

LLM делает:

P(token \mid context)

Разница только в масштабе.













