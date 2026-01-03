---
description: (2 признака)
---

# Кейс 3. Спам или не спам

Признаки\
– количество ссылок\
– длина письма

```
$samples = [
    [0, 50],
    [1, 120],
    [5, 300],
    [7, 500],
    [0, 40],
];

$labels = [0, 0, 1, 1, 0];

$dataset = new Labeled($samples, $labels);

$model = new LogisticRegression(0.01, 1000);
$model->train($dataset);

print_r($model->predict([[3, 200]]));
```

Что добавилось

Появляется линейная decision boundary в двумерном пространстве.
