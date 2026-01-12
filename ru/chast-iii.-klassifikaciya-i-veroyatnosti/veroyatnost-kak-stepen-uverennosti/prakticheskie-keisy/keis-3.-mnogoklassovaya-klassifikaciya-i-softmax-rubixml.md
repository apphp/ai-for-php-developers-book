# Кейс 3. Многоклассовая классификация и softmax (RubixML)

#### Сценарий

Письмо может относиться к нескольким категориям.

#### Модель

```
use Rubix\ML\Classifiers\SoftmaxClassifier;
use Rubix\ML\Classifiers\KNearestNeighbors;

$baseEstimator = new KNearestNeighbors(3);
$model = new SoftmaxClassifier($baseEstimator);

$model->train($dataset);
```

#### Вероятности

```
$probabilities = $model->proba($sample)[0];
print_r($probabilities);
```

#### Вывод

Softmax позволяет видеть распределение уверенности между всеми классами, а не только победителя.
