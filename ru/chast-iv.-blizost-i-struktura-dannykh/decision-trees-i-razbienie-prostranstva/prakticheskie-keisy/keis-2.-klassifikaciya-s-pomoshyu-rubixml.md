# Кейс 2. Классификация с помощью RubixML

Теперь посмотрим, как та же логика реализована в реальной библиотеке.

#### Подготовка данных

```php
use Rubix\ML\Datasets\Labeled;

$samples = [
    [5, 10],
    [7, 15],
    [1, 2],
    [2, 3],
    [6, 8],
    [3, 4],
];

$labels = ['active', 'active', 'passive', 'passive', 'active', 'passive'];

$dataset = new Labeled($samples, $labels);
```

#### Обучение дерева решений

```php
use Rubix\ML\Classifiers\ClassificationTree;

$estimator = new ClassificationTree(
    maxDepth: 3,
    minSamples: 2
);

$estimator->train($dataset);
```

#### Предсказание

```php
$prediction = $estimator->predict([[4, 6]]);

var_dump($prediction);
```

RubixML внутри делает всё то же самое: считает энтропию, information gain и строит дерево. Разница в том, что это уже оптимизированная, протестированная и расширяемая реализация.

#### Интерпретация результата

Хотя RubixML не всегда визуализирует дерево напрямую, сама структура остаётся объяснимой. Мы можем логически восстановить путь решения: какой признак был проверен первым, какие пороги оказались ключевыми.

###
