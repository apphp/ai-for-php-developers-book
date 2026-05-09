# Page 3. Base Rate Neglect в phishing-симуляциях

Это особенно интересно для B2B-продукта.

#### Сценарий

Ты анализируешь ответы сотрудников:

– 5% писем в системе – фишинговые

– 95% – нормальные

LLM классифицирует письмо как фишинг с уверенностью 90%.

Но игнорирует базовую вероятность.

Правильная формула:

P(Phish \mid Flag) = \frac{P(Flag \mid Phish) P(Phish)}{P(Flag)}

#### Реализация с RubixML (Naive Bayes)

```
<?php

use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Classifiers\NaiveBayes;

$samples = [
    ['urgent account suspension'],
    ['click here to reset password'],
    ['meeting agenda attached'],
    ['project deadline update'],
];

$labels = [
    'phish',
    'phish',
    'normal',
    'normal',
];

$dataset = new Labeled($samples, $labels);

$model = new NaiveBayes();
$model->train($dataset);

$prediction = $model->predict(['urgent password reset']);

print_r($prediction);
```

#### Что происходит математически

RubixML учитывает априорную вероятность классов.

LLM – часто нет.

#### Вывод для security-архитектуры

LLM можно использовать:

– для генерации объяснения сотруднику

– для перефразирования риска

– для создания персонализированной обратной связи

Но финальная классификация должна опираться на вероятностную модель с учётом base rate.
