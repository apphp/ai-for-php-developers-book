# Кейс 1. Учебное дерево решений на чистом PHP

#### Задача

Представим простую бизнес-задачу: нужно классифицировать пользователей как "активный" или "пассивный" на основе двух признаков:

– количество визитов в неделю – среднее время на сайте (в минутах)

Данные маленькие и игрушечные, но логика та же, что и в реальных системах.

#### Данные

Каждая запись – это \[visits, time, label]:

```php
$data = [
    [5, 10, 'active'],
    [7, 15, 'active'],
    [1, 2, 'passive'],
    [2, 3, 'passive'],
    [6, 8, 'active'],
    [3, 4, 'passive'],
];
```

#### Энтропия и information gain

Начнём с функций энтропии и information gain. Здесь важно не оптимизировать код, а сохранить читаемость.

```php
function entropy(array $labels): float
{
    $counts = array_count_values($labels);
    $total = count($labels);

    $entropy = 0.0;
    foreach ($counts as $count) {
        $p = $count / $total;
        $entropy -= $p * log($p, 2);
    }

    return $entropy;
}

function informationGain(array $parent, array $left, array $right): float
{
    $parentLabels = array_column($parent, 2);
    $leftLabels   = array_column($left, 2);
    $rightLabels  = array_column($right, 2);

    $hParent = entropy($parentLabels);
    $hLeft   = entropy($leftLabels);
    $hRight  = entropy($rightLabels);

    $total = count($parent);

    $weighted = (count($left) / $total) * $hLeft
              + (count($right) / $total) * $hRight;

    return $hParent - $weighted;
}
```

#### Поиск лучшего разбиения

Ограничимся бинарными разбиениями по одному признаку и одному порогу.

```php
function split(array $data, int $featureIndex, float $threshold): array
{
    $left = [];
    $right = [];

    foreach ($data as $row) {
        if ($row[$featureIndex] < $threshold) {
            $left[] = $row;
        } else {
            $right[] = $row;
        }
    }

    return [$left, $right];
}
```

Теперь можно перебрать возможные пороги и выбрать разбиение с максимальным information gain. Даже на этом простом этапе уже видно, как дерево "задаёт вопрос" к данным.

#### Что важно понять из кейса

Этот пример не про эффективность. Он про механику:

– дерево оценивает чистоту узлов – выбирает вопрос, уменьшающий неопределённость – рекурсивно повторяет процесс

Если вы понимаете этот код, вы понимаете суть decision tree.

###
