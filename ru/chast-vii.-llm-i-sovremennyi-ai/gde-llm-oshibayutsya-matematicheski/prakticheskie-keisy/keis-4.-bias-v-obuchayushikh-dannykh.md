# Кейс 4. Bias в обучающих данных

#### Сценарий

Ты обучаешь кастомную модель на логах своей компании.

Если 80% атак приходят из определённого региона, модель начнёт переоценивать этот фактор.

Математически:

\text{Bias} = \mathbb{E}\[\hat{y}] - y

```
<?php

use Rubix\ML\Classifiers\KNearestNeighbors;
use Rubix\ML\Datasets\Labeled;

$samples = [];
$labels = [];

// 90 нормальных
for ($i = 0; $i < 90; $i++) {
    $samples[] = [rand(0, 5)];
    $labels[] = 'normal';
}

// 10 атак
for ($i = 0; $i < 10; $i++) {
    $samples[] = [rand(6, 10)];
    $labels[] = 'attack';
}

$dataset = new Labeled($samples, $labels);

$model = new KNearestNeighbors(3);
$model->train($dataset);

print_r($model->predict([7]));
```

Если не балансировать данные, модель будет смещена.

#### Инженерное решение

– Oversampling

– Undersampling

– Class weights

– Отдельная оценка recall/precision

