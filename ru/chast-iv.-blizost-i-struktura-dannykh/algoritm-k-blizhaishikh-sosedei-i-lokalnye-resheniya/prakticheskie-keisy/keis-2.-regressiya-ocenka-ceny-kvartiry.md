# Кейс 2. Регрессия: оценка цены квартиры

### Кейс 2. Регрессия: оценка цены квартиры (pure PHP)

#### Сценарий

Мы оцениваем стоимость квартиры по двум признакам:

– площадь (м²) – расстояние до центра (км)

k-NN здесь используется как локальный регрессор.

#### Данные

```php
$dataset = [
    [[40, 12], 120000],
    [[50, 10], 150000],
    [[60, 8], 190000],
    [[70, 6], 240000],
    [[80, 5], 300000],
];

$query = [65, 7];
$k = 3;
```

#### Предсказание

```php
$distances = [];

foreach ($dataset as [$features, $price]) {
    $distances[] = [
        'distance' => euclideanDistance($features, $query),
        'price' => $price
    ];
}

usort($distances, fn($a, $b) => $a['distance'] <=> $b['distance']);
$neighbors = array_slice($distances, 0, $k);

$sum = 0;
foreach ($neighbors as $neighbor) {
    $sum += $neighbor['price'];
}

$prediction = $sum / $k;

echo "Estimated price: $prediction";
```

**Интуиция:** цена – это среднее по локальному району в пространстве признаков.

