# Кейс 6. Фрод или нормальная транзакция

Признаки

– сумма

– страна

– количество операций за день

```
$samples = [
    [50, 1, 2],
    [5000, 0, 15],
    [200, 1, 1],
    [7000, 0, 20],
];

$labels = [0, 1, 0, 1];

$model->train(new Labeled($samples, $labels));

print_r($model->predict([[3000, 0, 10]]));
```

Обсуждение

False negative дороже false positive.
