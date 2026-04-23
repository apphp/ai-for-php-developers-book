# MNIST: распознавание цифр через kNN (без обучения)

В предыдущем кейсе мы рассматривали распознавание цифр через Naive Bayes – вероятностную модель, которая обучается на данных и строит глобальное представление о классах.

Теперь посмотрим на ту же задачу с альтернативной точки зрения.

Здесь не будет обучения в привычном смысле. Мы не будем искать параметры, оптимизировать функцию потерь или строить параметрическую модель. Хотя формально этап обучения всё же есть, но он тривиален: модель просто запоминает обучающие данные без построения обобщающей функции.

Вместо этого мы сделаем гораздо более прямую вещь: будем сравнивать новое изображение с уже известными и искать похожие.

Это и есть алгоритм k-ближайших соседей.

#### Цель кейса

Этот кейс нужен, чтобы:

* показать, как работает распознавание без обучения
* закрепить геометрическую интерпретацию данных
* продемонстрировать kNN на реальной задаче
* противопоставить kNN вероятностным моделям

Главная идея: распознавание = поиск похожих примеров.

#### Сценарий

Как и прежде мы используем датасет MNIST – изображения рукописных цифр.

Чтобы упростить задачу:

* оставим только цифры 0 и 1
* нормализуем пиксели (делим на 255)
* используем небольшой набор данных

Каждое изображение:

* размер 28×28
* преобразуется в вектор из 784 чисел

Таким образом, каждая цифра становится точкой в 784-мерном пространстве

#### Реализация на чистом PHP

Для начала создадим класс KNearestNeighbors.&#x20;

<details>

<summary><strong>Class KNearestNeighbors</strong></summary>

```php
class KNearestNeighbors {

    /**
     * Training feature vectors.
     */
    private array $trainSamples = [];

    /**
     * Training labels aligned by index with `$trainSamples`.
     */
    private array $trainLabels = [];

    public function __construct(array $samples, array $labels) {
        $this->trainSamples = $samples;
        $this->trainLabels = $labels;
    }

    /**
     * Compute Euclidean distance between two vectors.
     */
    public function euclideanDistance(array $a, array $b): float {
        $sum = 0.0;

        foreach ($a as $i => $value) {
            $diff = $value - $b[$i];
            $sum += $diff * $diff;
        }

        return sqrt($sum);
    }

    /**
     * Predict a label for a single query vector.
     *
     * @param array    $query      Feature vector to classify
     * @param int      $k          Number of neighbors to vote (top-K)
     * @param int|null $trainLimit Optional cap on how many training rows are used
     *
     * @return int|string The winning label (depends on how labels were provided)
     */
    public function predict(array $query, int $k = 3, ?int $trainLimit = null): int|string {
        $trainLimit = min($trainLimit, count($this->trainSamples), count($this->trainLabels));

        $k = max(1, min($k, $trainLimit));

        $distances = [];

        for ($i = 0; $i < $trainLimit; $i++) {
            $distances[] = [
                'distance' => $this->euclideanDistance($this->trainSamples[$i], $query),
                'label' => $this->trainLabels[$i],
            ];
        }

        usort($distances, static fn ($a, $b) => $a['distance'] <=> $b['distance']);
        $neighbors = array_slice($distances, 0, $k);

        $votes = array_count_values(array_column($neighbors, 'label'));

        arsort($votes);

        return array_key_first($votes);
    }

    public function predictBatch(array $X, int $k = 3, ?int $trainLimit = null): array {
        $predictions = [];

        foreach ($X as $x) {
            $predictions[] = $this->predict($x, $k, $trainLimit);
        }

        return $predictions;
    }

    /**
     * Compute simple accuracy score for a labeled dataset.
     */
    public function score(array $X, array $y, int $k = 3, ?int $trainLimit = null): float {
        $predictions = $this->predictBatch($X, $k, $trainLimit);
        $correct = 0;

        foreach ($predictions as $i => $prediction) {
            if (isset($y[$i]) && $prediction === $y[$i]) {
                $correct++;
            }
        }

        return count($y) > 0 ? ($correct / count($y)) : 0.0;
    }
}

```

</details>

Теперь загрузим две наши подвыборки и запустим подсчёт `score`:

```php
try {
    [$trainSamples, $trainLabels] = MnistLoader::load('train.csv');
    [$testSamples, $testLabels] = MnistLoader::load('test.csv');
} catch (Exception $e) {
    echo '<div class="alert alert-danger" role="alert">' . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8') . '</div>';
    exit;
}

$model = new KNearestNeighbors($trainSamples, $trainLabels);

// Рассчитайте точность модели.
$score = $model->score($testSamples, $testLabels, k: 3, trainLimit: 300);

echo 'Обработано данных для обучения: ' . number_format(count($trainSamples)) . "\n";
echo 'Обработано данных для тестирования: ' . number_format(count($testSamples)) . "\n\n";
echo 'Точность: ' . round($score * 100, 2) . '%';
```

**Результат:**

```
Train samples handled: 12,666
Test samples handled: 2,116

Accuracy: 99.81%
```

**Что здесь происходит**

Обратите внимание: нет этапа обучения. Что касается такого высокого результата, то он возможен из-за упрощённой задачи (0 vs 1), небольшого размера выборки и конкретной подвыборки данных. На полном MNIST точность обычно немного ниже.

Алгоритм:

* просто хранит данные
* сравнивает новый объект со всеми остальными
* принимает решение на лету

Это ключевое отличие kNN.

**Интуиция**

Модель буквально делает следующее: "Найди изображения, которые выглядят максимально похоже, и посмотри, какие это цифры".

Это не "понимание цифры", а сравнение с примерами.

#### Реализация через RubixML

Теперь тот же подход через библиотеку:

```php
use app\classes\MnistLoader;
use Rubix\ML\CrossValidation\Metrics\Accuracy;
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Classifiers\KNearestNeighbors;
use Rubix\ML\Kernels\Distance\Euclidean;

try {
    // Build the training and test datasets from the filtered CSV rows.
    $trainRows = MnistLoader::loadIterable('train.csv', categoricalLabels: true, normalize: true, digits: [0, 1]);
    $testRows = MnistLoader::loadIterable('test.csv', categoricalLabels: true, normalize: true, digits: [0, 1]);

    $dataset = Labeled::fromIterator($trainRows);
    $testDataset = Labeled::fromIterator($testRows);
} catch (Exception $e) {
    echo '<div class="alert alert-danger" role="alert">' . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8') . '</div>';
    exit;
}

$model = new KNearestNeighbors(3, false, new Euclidean());
$model->train($dataset);

$predictions = [];
$testingLabels = $testDataset->labels();

foreach ($testDataset->samples() as $i => $x) {
    $prediction = $model->predict(new Unlabeled([$x]))[0];
    $predictions[] = $prediction;
}

$metric = new Accuracy();
$score = $metric->score($predictions, $testingLabels);

echo 'Обработано данных для обучения: ' . number_format($dataset->numSamples()) . "\n";
echo 'Обработано данных для тестирования: ' . number_format($testDataset->numSamples()) . "\n\n";
echo 'Точность: ' . round($score * 100, 2) . '%';
```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Точность: 99.92%
```

**Объяснение:**

Здесь важно понимать: `train()` не обучает модель в привычном смысле – он просто сохраняет датасет без построения обобщающей зависимости.

Что изменилось, а что нет.  Код стал короче, но логика не изменилась.

Внутри происходит всё то же самое:

* считаются расстояния
* выбираются ближайшие изображения
* выполняется голосование

Разница только в том, что библиотека берёт на себя: оптимизацию, работу с памятью, удобный API.

**Сравнение с Naive Bayes**

Теперь контраст становится очевидным.

Naive Bayes:

* строит вероятностную модель
* делает глобальные предположения
* "учится" на данных

kNN:

* не обучается
* не строит параметрическую модель
* работает через сравнение

#### Ограничения

MNIST хорошо показывает и слабые стороны kNN:

* высокая размерность (784 признака)
* медленная работа при больших данных (каждое предсказание требует O(n) сравнений)
* чувствительность к метрике и масштабам

#### Выводы и следующие шаги

Этот кейс показывает один из самых важных и, на первый взгляд, неожиданных взглядов на машинное обучение: иногда модель – это просто данные.

В kNN нет привычного этапа обучения. Мы не подбираем коэффициенты, не оптимизируем функцию потерь, не строим явную модель. Вместо этого мы сохраняем примеры и каждый раз принимаем решение, опираясь на их локальное окружение.&#x20;

В этом смысле kNN предельно "честен":

* он не скрывает логику за параметрами
* не делает сильных предположений о данных
* и не пытается обобщить всё одной формулой

Всё, что у него есть – это расстояния и соседи. И этого уже достаточно, чтобы решать реальные задачи.

Именно поэтому kNN так важен для понимания. Он убирает слой абстракции и показывает суть: машинное обучение – это работа со структурой данных, а не скрытая логика работы моделей.

При этом становится заметно и ограничение такого подхода. Когда данных становится много, а размерность растёт, "простая" стратегия перебора соседей становится вычислительно дорогой и может ухудшаться по качеству из-за высокой размерности данных.

Это естественным образом подводит нас к следующему шагу.

В следующей главе мы рассмотрим другой подход – деревья решений. В отличие от kNN, они строят явную структуру, которая позволяет быстро принимать решения и лучше масштабируется, но при этом сохраняет интерпретируемость.

**Прогресс**

Мы будем возвращаться к этому кейсу в следующих главах и постепенно улучшать модель, добавляя новые идеи и методы. Важно не просто увидеть разные алгоритмы, а понять, как они меняют поведение модели на одной и той же задаче. Ниже – прогресс, который мы имеет на текущий момент.

Прогресс моделей на MNIST:

<table><thead><tr><th width="193.01953125">Модель</th><th width="181.01171875">Задача</th><th width="168.92578125">Точность</th><th>Комментарий</th></tr></thead><tbody><tr><td><a href="../../../chast-iii.-klassifikaciya-i-veroyatnosti/logisticheskaya-regressiya/skvoznoi-keis-raspoznavanie-cifr-mnist/mnist-binarnaya-klassifikaciya-otlichaem-0-ot-1.md">Logistic</a></td><td>0 vs 1 (binary)</td><td>~99%</td><td>линейная</td></tr><tr><td><a href="../../../chast-iii.-klassifikaciya-i-veroyatnosti/pochemu-naivnyi-baies-rabotaet/skvoznoi-keis-raspoznavanie-cifr-mnist/mnist-veroyatnostnaya-model-klassifikacii-cifr-naive-bayes.md">Naive Bayes</a></td><td>0 vs 1 (binary)</td><td>~98–99%</td><td>простая вероятностная модель, предполагает независимость пикселей</td></tr><tr><td><a href="./">kNN</a></td><td>0 vs 1 (binary)</td><td>~99%</td><td>локальная модель, основанная на расстояниях, без обучения</td></tr><tr><td>...</td><td></td><td>...</td><td>...</td></tr></tbody></table>

{% hint style="info" %}
Чтобы самостоятельно протестировать этот код, воспользуйтесь [онлайн-демонстрацией](https://aiwithphp.org/books/ai-for-php-developers/examples/part-4/k-nearest-neighbors-algorithm-and-local-solutions) для его запуска.
{% endhint %}
