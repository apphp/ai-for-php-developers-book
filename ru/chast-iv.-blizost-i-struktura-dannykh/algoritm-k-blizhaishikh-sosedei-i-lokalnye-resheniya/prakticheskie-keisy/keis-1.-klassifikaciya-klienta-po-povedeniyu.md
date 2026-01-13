# Кейс 1. Классификация клиента по поведению

### Кейс 1. Классификация клиента по поведению (pure PHP)

#### Сценарий

Есть простая модель поведения пользователей сайта. Для каждого пользователя мы знаем:

– количество посещений за месяц – среднее время на сайте (в минутах)

Наша задача – классифицировать нового пользователя как **"engaged"** или **"casual"**.

#### Подготовка данных

```php
<?php
$dataset = [
    [[5, 2.1], 'casual'],
    [[3, 1.8], 'casual'],
    [[10, 6.5], 'engaged'],
    [[12, 7.0], 'engaged'],
    [[9, 5.8], 'engaged'],
];

$query = [8, 5.5]; // новый пользователь
$k = 3;
```

#### Функция расстояния (евклидова)

```php
function euclideanDistance(array $a, array $b): float
{
    $sum = 0.0;
    foreach ($a as $i => $value) {
        $sum += ($value - $b[$i]) ** 2;
    }
    return sqrt($sum);
}
```

#### Поиск k ближайших соседей

```php
$distances = [];

foreach ($dataset as [$point, $label]) {
    $distances[] = [
        'distance' => euclideanDistance($point, $query),
        'label' => $label
    ];
}

usort($distances, fn($a, $b) => $a['distance'] <=> $b['distance']);
$neighbors = array_slice($distances, 0, $k);
```

#### Голосование

```php
$votes = [];
foreach ($neighbors as $neighbor) {
    $votes[$neighbor['label']] = ($votes[$neighbor['label']] ?? 0) + 1;
}

arsort($votes);
$prediction = array_key_first($votes);

echo "Prediction: $prediction";
```

**Интуиция:** решение принимается строго локально – только по трём ближайшим пользователям.

