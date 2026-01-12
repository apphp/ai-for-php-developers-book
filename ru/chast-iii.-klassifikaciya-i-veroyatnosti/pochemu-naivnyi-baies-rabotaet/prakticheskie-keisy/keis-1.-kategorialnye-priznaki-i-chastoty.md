---
description: Понять механику
---

# Кейс 1. Категориальные признаки и частоты

## Наивный Байес из частот (чистая логика)

#### Сценарий

Мы классифицируем пользователя как:

* buyer
* browser

На основе двух признаков:

* from\_ads (пришел из рекламы)
* has\_account (есть аккаунт)

### Реализация: чистый PHP

```
<?php

$data = [
    ['class' => 'buyer',   'from_ads' => true,  'has_account' => true],
    ['class' => 'buyer',   'from_ads' => false, 'has_account' => true],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
];

// Подсчет априорных вероятностей
$classCounts = [];
$featureCounts = [];

foreach ($data as $row) {
    $class = $row['class'];
    $classCounts[$class] = ($classCounts[$class] ?? 0) + 1;

    foreach ($row as $feature => $value) {
        if ($feature === 'class') continue;
        $featureCounts[$class][$feature][$value] =
            ($featureCounts[$class][$feature][$value] ?? 0) + 1;
    }
}

// Классификация нового объекта
$input = ['from_ads' => true, 'has_account' => true];

$scores = [];

foreach ($classCounts as $class => $count) {
    $logProb = log($count / count($data));

    foreach ($input as $feature => $value) {
        $featureCount = $featureCounts[$class][$feature][$value] ?? 1;
        $total = $classCounts[$class];
        $logProb += log($featureCount / $total);
    }

    $scores[$class] = $logProb;
}

arsort($scores);
print_r($scores);
```

#### Что важно понять из этого кейса

* Никакого обучения в привычном смысле нет
* Только частоты и логарифмы
* Это тот же код, что и формулы в главе

### Та же идея: RubixML

```
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Classifiers\NaiveBayes;

$samples = [
    [1, 1],
    [0, 1],
    [1, 0],
    [1, 0],
];

$labels = ['buyer', 'buyer', 'browser', 'browser'];

$dataset = new Labeled($samples, $labels);

$model = new NaiveBayes();
$model->train($dataset);

$prediction = $model->predict([[1, 1]]);
print_r($prediction);
```

