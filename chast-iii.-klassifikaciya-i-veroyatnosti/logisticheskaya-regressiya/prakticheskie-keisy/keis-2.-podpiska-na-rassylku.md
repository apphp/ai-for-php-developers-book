---
description: (1 признак)
---

# Кейс 2. Подписка на рассылку

Суть

Предсказать, подпишется ли пользователь на рассылку по времени, проведенному на сайте.

Признак

– время на сайте (минуты)

```
use Rubix\ML\Classifiers\LogisticRegression;
use Rubix\ML\Datasets\Labeled;

$samples = [
    [0.5],
    [1.2],
    [2.0],
    [5.0],
    [7.0],
];

$labels = [0, 0, 0, 1, 1];

$dataset = new Labeled($samples, $labels);

$model = new LogisticRegression(0.1, 500);
$model->train($dataset);

$result = $model->predict([[3.0]]);
print_r($result);
```

Что здесь важно

Это буквально сигмоида вдоль одной оси. Decision boundary – точка, где вероятность равна 0.5.

