# MNIST: вероятностная модель классификации цифр (Naive Bayes)

#### Naive Bayes как следующий шаг после бинарной классификации

В предыдущем кейсе мы решали упрощенную задачу: отличали 0 от 1. Это позволило сфокусироваться на базовой механике модели. Теперь мы делаем следующий шаг – переходим к полноценной многоклассовой классификации и рассматриваем задачу шире.

MNIST – это классический датасет рукописных цифр. Каждое изображение – это сетка пикселей, а задача модели – определить, какая цифра на нем изображена.

Наивный Байес здесь выступает как вероятностная модель, которая оценивает, насколько "похоже" изображение на каждый класс цифр.

#### Цель кейса

Показать, как:

* наивный Байес применяется к задаче компьютерного зрения
* изображение превращается в набор числовых признаков
* модель оценивает вероятность каждого класса
* происходит переход от бинарной классификации к многоклассовой

Этот кейс – логическое продолжение предыдущего и завершает линию "от простой задачи к более реалистичной".

#### Сценарий

Мы классифицируем изображение рукописной цифры:

* вход: изображение 28×28 пикселей
* выход: класс от 0 до 9

Каждый пиксель – это число от 0 до 255.

Перед подачей в модель мы нормализуем значения:

> делим на 255 → получаем диапазон \[0, 1]

Таким образом, одно изображение превращается в вектор из 784 числовых признаков.

#### Идея модели

Для каждого класса (цифры 0–9) наивный Байес оценивает:

* среднее значение каждого пикселя
* дисперсию каждого пикселя

Далее для нового изображения считается:

$$
P(\text{class} \mid \text{image}) \propto P(\text{class}) \cdot \prod_i P(\text{pixel}_i \mid \text{class})
$$

Где $$P(pixelᵢ | class)$$ задается через нормальное распределение.

#### Упрощение для кейса

Чтобы код оставался понятным, используем:

* только цифры 0 и 1 (связь с предыдущим кейсом)
* нормализацию пикселей

#### Реализация на чистом PHP

Для начала создадим класс Naive Bayes.&#x20;

<details>

<summary><strong>Class GaussianNB</strong></summary>

```php
class GaussianNB {
    private array $stats = [];
    private array $grouped = [];
    private int $totalSamples = 0;

    // Calculate arithmetic mean of values
    private function mean(array $values): float {
        return array_sum($values) / count($values);
    }

    // Calculate variance of values given their mean
    private function variance(array $values, float $mean): float {
        $sum = 0;
        foreach ($values as $v) {
            $sum += pow($v - $mean, 2);
        }
        return $sum / count($values);
    }

    // Calculate Gaussian probability density function
    private function gaussian(float $x, float $mean, float $variance): float {
        return (1 / sqrt(2 * pi() * $variance)) * exp(-pow($x - $mean, 2) / (2 * $variance));
    }

    // Train the classifier by calculating mean and variance for each class/feature
    public function train(array $X, array $y): void {
        $this->totalSamples = count($X);

        // Group samples by class
        $this->grouped = [];
        foreach ($X as $i => $sample) {
            $this->grouped[$y[$i]][] = $sample;
        }

        // Calculate statistics for each class and feature
        $this->stats = [];
        foreach ($this->grouped as $class => $rows) {
            $features = array_map(null, ...$rows);

            foreach ($features as $i => $values) {
                $m = $this->mean($values);
                $v = $this->variance($values, $m);

                $this->stats[$class][$i] = [
                    'mean' => $m,
                    'variance' => $v ?: 1e-6,
                ];
            }
        }
    }

    // Predict class labels for multiple samples
    public function predict(array $X): array {
        $predictions = [];

        foreach ($X as $sample) {
            $predictions[] = $this->predictSingle($sample);
        }

        return $predictions;
    }

    // Predict class label for a single sample using Bayes' theorem
    public function predictSingle(array $input): int {
        $scores = [];

        foreach ($this->stats as $class => $features) {
            // Start with prior probability (class frequency)
            $logProb = log(count($this->grouped[$class]) / $this->totalSamples);

            // Add likelihood for each feature (naive independence assumption)
            foreach ($features as $i => $params) {
                $prob = $this->gaussian($input[$i], $params['mean'], $params['variance']);
                $logProb += log($prob);
            }

            $scores[$class] = $logProb;
        }

        // Return class with highest score
        arsort($scores);
        return (int) array_key_first($scores);
    }

    // Calculate accuracy of the classifier on test data
    public function accuracy(array $X, array $y): float {
        $predictions = $this->predict($X);
        $correct = 0;

        foreach ($predictions as $i => $prediction) {
            if ($prediction === $y[$i]) {
                $correct++;
            }
        }

        return count($y) > 0 ? ($correct / count($y)) : 0.0;
    }

    // Get calculated statistics (mean and variance) for each class/feature
    public function getStats(): array {
        return $this->stats;
    }

    // Get grouped training data by class
    public function getGrouped(): array {
        return $this->grouped;
    }
}

```

</details>

Теперь загрузим две наши подвыборки&#x20;

<details>

<summary><strong>Class MnistLoader</strong></summary>

```php
class MnistLoader {

    static public function loadMNIST(string $file, string $directory = '', bool $categoricalLabels = false): array {
        $X = [];
        $y = [];

        $handle = fopen($directory . $file, 'r');

        if ($handle === false) {
            throw new Exception('File not found.');
        }

        while (($row = fgetcsv($handle)) !== false) {
            if ($row === [] || $row[0] === null || $row[0] === '') {
                continue;
            }

            $label = (int)$row[0];

            // 1. Оставляем только 0 и 1
            if ($label !== 0 && $label !== 1) {
                continue;
            }

            $pixels = array_slice($row, 1);

            // 2. Нормализация (0–255 → 0–1)
            $pixels = array_map(function ($p): float {
                return ((float) trim((string) $p)) / 255.0;
            }, $pixels);

            $X[] = $pixels;
            $y[] = $categoricalLabels
                ? ($label === 1 ? 'one' : 'zero')
                : $label;
        }

        return [$X, $y];
    }
}

```

</details>

И начнём и тренировать:

```php
try {
    [$trainSamples, $trainLabels] = MnistLoader::loadMNIST('train.csv');;
    [$testSamples, $testLabels] = MnistLoader::loadMNIST('test.csv');
} catch (Exception $e) {
    echo '<div class="alert alert-danger" role="alert">' . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8') . '</div>';
    exit;
}

// Initialize and train the Gaussian Naive Bayes classifier
$model = new GaussianNB();
$model->train($trainSamples, $trainLabels);

// Calculate model accuracy
$accuracy = $model->accuracy($testSamples, $testLabels);

echo 'Train samples handled: ' . number_format(count($trainSamples)) . "\n";
echo 'Test samples handled: ' . number_format(count($testSamples)) . "\n\n"
echo 'Accuracy: ' . round($accuracy * 100, 2) . '%';
```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Точность: 98.58%
```

**Объяснение:**

Модель выбирает класс, для которого изображение наиболее вероятно.

**Что здесь происходит интуитивно**

Каждый пиксель задает вопрос:

> "Насколько это значение похоже на то, что я обычно вижу у этой цифры?"

Если большинство пикселей "согласны" с классом – он побеждает.

Это снова то же самое голосование признаков, только теперь признаков сотни.

**Связь с предыдущим кейсом (0 vs 1)**

Здесь происходит важный переход:

* было: один бинарный классификатор
* стало: несколько классов (многоклассовая модель)

Но алгоритм не изменился:

* те же вероятности
* та же независимость
* та же логика выбора максимума

**Ограничения модели на MNIST**

Наивный Байес здесь работает, но:

* пиксели явно зависимы (контуры, формы)
* модель не учитывает структуру изображения
* качество ниже, чем у нейросетей

Тем не менее:

* модель обучается мгновенно
* работает быстро
* дает разумный baseline

#### Та же задача с использованием RubixML

```php
use Rubix\ML\Classifiers\GaussianNB;
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Extractors\CSV;

try {
    function mnistRows(string $file): iterable {
        // Read the CSV file row by row and keep only valid samples.
        foreach (new CSV($file, false) as $row) {
            if (!isset($row[0])) {
                continue;
            }

            // The first column is the digit label.
            $label = (int) $row[0];

            // This example only trains on digits 0 and 1.
            if ($label !== 0 && $label !== 1) {
                continue;
            }

            // Normalize pixel values to the [0, 1] range for training.
            $pixels = array_map(static fn ($value): float => ((float) $value) / 255.0, array_slice($row, 1));

            // Rubix expects the features followed by the class label.
            yield array_merge($pixels, [$label === 1 ? 'one' : 'zero']);
        }
    }

    // Build the training and test datasets from the filtered CSV rows.
    $trainRows = mnistRows('train.csv');
    $testRows = mnistRows('test.csv');

    $dataset = Labeled::fromIterator($trainRows);
    $testDataset = Labeled::fromIterator($testRows);
} catch (Exception $e) {
    echo '<div class="alert alert-danger" role="alert">' . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8') . '</div>';
    exit;
}

$model = new GaussianNB();
$model->train($dataset);

$correct = 0;

foreach ($testDataset->samples() as $i => $x) {
    $prediction = $model->predict(new Unlabeled([$x]))[0];

    if ($prediction === $testDataset->labels()[$i]) {
        $correct++;
    }
}

$accuracy = $correct / $testDataset->numSamples();

echo 'Train samples handled: ' . number_format($dataset->numSamples()) . PHP_EOL;
echo 'Test samples handled: ' . number_format($testDataset->numSamples()) . PHP_EOL . PHP_EOL;
echo 'Accuracy: ' . round($accuracy * 100, 2) . '%';
```

```php
```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Точность: 99.15%
```

**Объяснение:**

RubixML делает то же самое:

* считает средние и дисперсии по каждому пикселю
* применяет Gaussian Naive Bayes
* возвращает наиболее вероятный класс

#### Вывод и следующие шаги

Этот кейс завершает линию развития:

* от простых признаков → к тексту → к числовым данным → к изображениям
* от бинарной классификации → к многоклассовой

Наивный Байес показывает здесь свою главную силу:

> он масштабируется по количеству признаков почти без усложнения логики.

Несмотря на сильное предположение о независимости пикселей, модель все равно способна различать цифры – потому что даже приближенные вероятности дают достаточно информации для правильного выбора.

И это один из самых важных уроков машинного обучения: идеальная модель не всегда нужна – достаточно той, которая правильно ранжирует варианты.

Наивный Байес даёт быстрый, но довольно грубый результат из-за сильного предположения независимости признаков. Чтобы учесть реальную структуру данных, в следующей главе мы попробуем более гибкий подход – k ближайших соседей (kNN), который опирается на схожесть изображений и позволяет моделировать нелинейные границы.

**Прогресс**

Мы продолжим возвращаться к этому кейсу в следующих главах и постепенно улучшать модель. Ниже – прогресс, который мы имеет на текущий момент.

Прогресс моделей на MNIST:

<table><thead><tr><th width="193.01953125">Модель</th><th width="181.01171875">Задача</th><th width="168.92578125">Точность</th><th>Комментарий</th></tr></thead><tbody><tr><td><a href="../../logisticheskaya-regressiya/skvoznoi-keis-raspoznavanie-cifr-mnist/mnist-binarnaya-klassifikaciya-otlichaem-0-ot-1.md">Logistic</a></td><td>0 vs 1 (binary)</td><td>~99%</td><td>линейная</td></tr><tr><td><a href="mnist-veroyatnostnaya-model-klassifikacii-cifr-naive-bayes.md">Naive Bayes</a></td><td>0 vs 1 (binary)</td><td>~98–99%</td><td>простая вероятностная модель, предполагает независимость пикселей</td></tr><tr><td>...</td><td></td><td>...</td><td>...</td></tr></tbody></table>
