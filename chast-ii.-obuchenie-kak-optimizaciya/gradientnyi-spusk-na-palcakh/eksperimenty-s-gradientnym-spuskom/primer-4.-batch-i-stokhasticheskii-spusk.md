# 🟡 Пример 4. Batch и стохастический спуск

Последний кейс – сравнение ровного и шумного обучения.

**Batch gradient descent**

```php
$x = [1, 2, 3, 4];
$y = [2, 4, 6, 8];

$w = 0.0;
$lr = 0.1;
$n = count($x);

echo "Batch GD" . PHP_EOL;

for ($epoch = 1; $epoch <= 10; $epoch++) {

    $gradient = 0.0;

    for ($i = 0; $i < $n; $i++) {
        $gradient += $x[$i] * (($w * $x[$i]) - $y[$i]);
    }

    $gradient = (2 / $n) * $gradient;
    $w -= $lr * $gradient;

    echo "Epoch $epoch: w = " . round($w, 4) . PHP_EOL;
}
```

**Stochastic gradient descent**

```php
$x = [1, 2, 3, 4];
$y = [2, 4, 6, 8];

$w = 0.0;
$lr = 0.1;
$n = count($x);

echo PHP_EOL . "Stochastic GD" . PHP_EOL;

for ($epoch = 1; $epoch <= 10; $epoch++) {

    for ($i = 0; $i < $n; $i++) {
        $gradient = 2 * $x[$i] * (($w * $x[$i]) - $y[$i]);
        $w -= $lr * $gradient;
    }

    echo "Epoch $epoch: w = " . round($w, 4) . PHP_EOL;
}
```

Что читатель видит:

– batch идет гладко и предсказуемо;

– SGD дергается, шумит, но часто сходится быстрее;

– шум – не баг, а свойство метода.





