# Кейс 4. Клик по рекламе (CTR)

Признаки

– время суток

– тип устройства (0 – desktop, 1 – mobile)

```
$samples = [
    [9, 0],
    [12, 1],
    [18, 1],
    [22, 0],
    [14, 1],
];

$labels = [0, 1, 1, 0, 1];

$dataset = new Labeled($samples, $labels);

$model = new LogisticRegression(0.05, 800);
$model->train($dataset);

print_r($model->predict([[20, 1]]));
```

Смысл

Логистическая регрессия – стандарт де-факто для CTR.
