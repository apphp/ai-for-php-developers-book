# Кейс 5. Обучение модели как минимизация ошибки

### Обучение «в лоб»: минимизация loss вручную

Сценарий\
Мы обучаем линейную модель y = wx, перебирая значения w и считая MSE.

Цель кейса

Связать loss-функцию и обучение без градиентного спуска – интуитивно.

```
$x = [1, 2, 3, 4];
$y = [2, 4, 6, 8];

function predict(array $x, float $w): array {
    return array_map(fn($xi) => $w * $xi, $x);
}

$bestW = null;
$bestLoss = INF;

for ($w = 0; $w <= 3; $w += 0.01) {
    $yHat = predict($x, $w);
    $loss = mse($y, $yHat);

    if ($loss < $bestLoss) {
        $bestLoss = $loss;
        $bestW = $w;
    }
}

echo "Best w = $bestW, loss = $bestLoss";
```

Почему это важно

Читатель буквально видит:

минимизация loss = обучение модели.



### То же самое, но с библиотекой (Rubix ML)

Сценарий

Показать, что библиотеки делают то же самое, но скрывают детали.

```
use Rubix\ML\Regressors\LeastSquares;
use Rubix\ML\Datasets\Labeled;

$samples = [[1], [2], [3], [4]];
$labels = [2, 4, 6, 8];

$dataset = new Labeled($samples, $labels);

$model = new LeastSquares();
$model->train($dataset);

$predictions = $model->predict([[5]]);
print_r($predictions);
```

Связь с главой

Под капотом происходит минимизация MSE, даже если мы её не видим.
