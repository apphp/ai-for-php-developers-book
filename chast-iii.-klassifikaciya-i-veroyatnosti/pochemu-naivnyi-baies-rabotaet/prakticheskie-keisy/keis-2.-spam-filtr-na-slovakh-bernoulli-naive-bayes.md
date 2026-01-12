---
description: Связать с реальным ML
---

# Кейс 2. Спам-фильтр на словах (Bernoulli Naive Bayes)

## Спам-фильтр по словам (Bernoulli Naive Bayes)

#### Сценарий

Классический спам-фильтр:

* слова либо есть, либо нет
* частоты не важны

### Реализация: чистый PHP

```
<?php

$emails = [
    ['text' => ['free', 'win'], 'class' => 'spam'],
    ['text' => ['free'],        'class' => 'spam'],
    ['text' => ['meeting'],     'class' => 'ham'],
    ['text' => ['project'],     'class' => 'ham'],
];

$classes = [];
$wordCounts = [];
$totalDocs = count($emails);

foreach ($emails as $email) {
    $class = $email['class'];
    $classes[$class] = ($classes[$class] ?? 0) + 1;

    foreach ($email['text'] as $word) {
        $wordCounts[$class][$word] =
            ($wordCounts[$class][$word] ?? 0) + 1;
    }
}

$input = ['free', 'meeting'];
$scores = [];

foreach ($classes as $class => $count) {
    $logProb = log($count / $totalDocs);

    foreach ($input as $word) {
        $wordCount = $wordCounts[$class][$word] ?? 1;
        $logProb += log($wordCount / $count);
    }

    $scores[$class] = $logProb;
}

arsort($scores);
print_r($scores);
```

#### Связь с главой

* Каждое слово – независимое доказательство
* Это буквально раздел «Голосование признаков»
* Лог-суммирование = линейная модель

### Та же задача: RubixML

```
use Rubix\ML\Classifiers\NaiveBayes;
use Rubix\ML\Datasets\Labeled;

$samples = [
    [1, 1, 0], // free, win, meeting
    [1, 0, 0],
    [0, 0, 1],
    [0, 0, 0],
];

$labels = ['spam', 'spam', 'ham', 'ham'];

$dataset = new Labeled($samples, $labels);

$model = new NaiveBayes();
$model->train($dataset);

$prediction = $model->predict([[1, 0, 1]]);
print_r($prediction);
```

