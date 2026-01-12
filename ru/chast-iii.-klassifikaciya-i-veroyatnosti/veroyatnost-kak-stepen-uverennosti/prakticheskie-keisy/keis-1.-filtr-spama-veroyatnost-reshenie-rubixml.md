# Кейс 1. Фильтр спама: вероятность ≠ решение (RubixML)

#### Сценарий

Модель классифицирует письма как спам или не спам и возвращает вероятность. Решение о том, что делать с письмом, принимается отдельно.

#### Модель и данные

```
use Rubix\ML\Classifiers\LogisticRegression;
use Rubix\ML\Datasets\Labeled;

$samples = [
    [3, 1],  // короткая тема, мало ссылок
    [15, 8], // длинная тема, много ссылок
    [5, 0],
];

$labels = ['normal', 'spam', 'normal'];

$dataset = new Labeled($samples, $labels);

$model = new LogisticRegression();
$model->train($dataset);
```

#### Предсказание вероятности

```
$sample = [[12, 6]]; // новое письмо
$probabilities = $model->proba($sample)[0];
```

#### Решение

```
$threshold = 0.7;

if ($probabilities['spam'] >= $threshold) {
    echo 'Move to spam';
} else {
    echo 'Keep in inbox';
}
```

#### Вывод

Модель оценивает уверенность. Порог – это инженерное решение, зависящее от цены ошибки.
