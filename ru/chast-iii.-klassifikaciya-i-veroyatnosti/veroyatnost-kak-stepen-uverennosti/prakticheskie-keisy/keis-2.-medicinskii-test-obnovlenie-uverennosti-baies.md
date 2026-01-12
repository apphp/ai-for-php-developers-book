# Кейс 2. Медицинский тест: обновление уверенности (Байес)

#### Сценарий

Вероятность болезни пересчитывается после получения результата теста.

#### Реализация

```
$prior = 0.001;
$sensitivity = 0.99;
$specificity = 0.95;

$falsePositive = 1 - $specificity;

$posterior = ($sensitivity * $prior)
    / (($sensitivity * $prior) + ($falsePositive * (1 - $prior)));

echo round($posterior, 4);
```

#### Вывод

Даже очень точный тест не дает высокой уверенности, если само событие редкое.
