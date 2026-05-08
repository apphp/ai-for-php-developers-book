# MNIST: вероятностная модель классификации цифр (Naive Bayes)

#### Naive Bayes как следующий шаг после бинарной классификации

В предыдущем кейсе мы решали упрощенную задачу: отличали 0 от 1. Это позволило сфокусироваться на базовой механике модели. Теперь мы делаем следующий шаг – переходим к полноценной многоклассовой классификации и рассматриваем задачу шире.

Однако в реализации ниже мы по-прежнему используем только цифры 0 и 1, чтобы сохранить преемственность с предыдущим кейсом. При этом сама модель уже обобщается на произвольное число классов.

Напомним, что MNIST – это классический датасет рукописных цифр. Каждое изображение – это сетка пикселей, а задача модели – определить, какая цифра на нем изображена.

Наивный Байес здесь выступает как вероятностная модель, которая оценивает вероятность того, что изображение принадлежит каждому классу цифр.

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
* выход: класс от 0 до 9 (в общем случае)

Каждый пиксель – это число от 0 до 255.

Перед подачей в модель мы нормализуем значения:

> делим на 255 → получаем диапазон \[0, 1]

Таким образом, одно изображение превращается в вектор из 784 числовых признаков. Это упрощает вычисления, хотя и не делает распределение пикселей нормальным (гауссовым), как предполагает модель Gaussian Naive Bayes.

#### Идея модели

Для каждого класса (цифры 0–9) наивный Байес оценивает:

* среднее значение каждого пикселя
* дисперсию каждого пикселя

Далее для нового изображения считается:

$$
P(\text{class} \mid \text{image}) \propto P(\text{class}) \cdot \prod_i P(\text{pixel}_i \mid \text{class})
$$

Где $$P(pixelᵢ \mid class)$$ задается через нормальное распределение.

На практике произведение вероятностей заменяется суммой логарифмов, чтобы избежать численного переполнения:

* произведение → сумма логарифмов
* маленькие числа не "схлопываются" в ноль

Именно поэтому в коде далее используется функция `log()`.

#### Упрощение для кейса

Чтобы код оставался понятным, используем:

* только цифры 0 и 1 (как и в предыдущем кейсе)
* нормализацию пикселей

При этом важно понимать, что модель в таком виде напрямую обобщается на все 10 классов (0–9).

#### Реализация на чистом PHP

Для начала создадим класс GaussianNB.&#x20;

<details>

<summary><strong>Class GaussianNB</strong></summary>

```php
class GaussianNB {
    private array $stats = [];
    private array $grouped = [];
    private int $totalSamples = 0;

    // Вычислим среднее арифметическое значений
    private function mean(array $values): float {
        return array_sum($values) / count($values);
    }

    // Вычислим дисперсию значений, зная их среднее значение
    private function variance(array $values, float $mean): float {
        $sum = 0;
        foreach ($values as $v) {
            $sum += pow($v - $mean, 2);
        }
        return $sum / count($values);
    }

    // Вычислим гауссову функцию плотности вероятности.
    private function gaussian(float $x, float $mean, float $variance): float {
        return (1 / sqrt(2 * pi() * $variance)) * exp(-pow($x - $mean, 2) / (2 * $variance));
    }

    // Обучим классификатор, вычислив среднее значение и дисперсию для каждого класса/признака
    public function train(array $X, array $y): void {
        $this->totalSamples = count($X);

        // Сгруппируем образцы по классам.
        $this->grouped = [];
        foreach ($X as $i => $sample) {
            $this->grouped[$y[$i]][] = $sample;
        }

        // Рассчитаем статистические данные для каждого класса и признака
        $this->stats = [];
        foreach ($this->grouped as $class => $rows) {
            // Чтобы получить массивы значений, отображающие отдельные признаки, 
            // преобразуем строки в столбцы
            $features = array_map(null, ...$rows);

            foreach ($features as $i => $values) {
                $m = $this->mean($values);
                $v = $this->variance($values, $m);

                $this->stats[$class][$i] = [
                    'mean' => $m,
                    // защита от нулевой дисперсии
                    'variance' => $v ?: 1e-6,
                ];
            }
        }
    }

    // Прогнозирование меток классов для нескольких образцов
    public function predict(array $X): array {
        $predictions = [];

        foreach ($X as $sample) {
            $predictions[] = $this->predictSingle($sample);
        }

        return $predictions;
    }

    // Предсказывание метки класса для одного образца, используя теорему Байеса
    public function predictSingle(array $input): int {
        $scores = [];

        foreach ($this->stats as $class => $features) {
            // Начнём с априорной вероятности (частоты класса).
            $logProb = log(count($this->grouped[$class]) / $this->totalSamples);

            // Добавим вероятность для каждого признака (наивное предположение о независимости).
            foreach ($features as $i => $params) {
                $prob = $this->gaussian($input[$i], $params['mean'], $params['variance']);
                $logProb += log($prob);
            }

            $scores[$class] = $logProb;
        }

        // Возвращение класса с наивысшим баллом
        arsort($scores);
        return (int) array_key_first($scores);
    }

    // Вычислим оценку для набора прогнозов
    public function score(array $X, array $y): float {
        $predictions = $this->predict($X);
        $correct = 0;

        foreach ($predictions as $i => $prediction) {
            if ($prediction === $y[$i]) {
                $correct++;
            }
        }

        return count($y) > 0 ? ($correct / count($y)) : 0.0;
    }

    // Получим вычисленные статистические данные (среднее значение и дисперсию) для каждого класса/признака
    public function getStats(): array {
        return $this->stats;
    }

    // Получим сгруппированных обучающих данных по классам.
    public function getGrouped(): array {
        return $this->grouped;
    }
}
```

</details>

Теперь загрузим две наши подвыборки и начнём тренировать:

```php
try {
    [$trainSamples, $trainLabels] = MnistLoader::load('train.csv');
    [$testSamples, $testLabels] = MnistLoader::load('test.csv');
} catch (Exception $e) {
    echo '<div class="alert alert-danger" role="alert">' . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8') . '</div>';
    exit;
}

// Инициализация и обучение гауссова наивного байесовского классификатора.
$model = new GaussianNB();
$model->train($trainSamples, $trainLabels);

// Рассчитайте точность модели.
$score = $model->score($testSamples, $testLabels);

echo 'Обработано данных для обучения: ' . number_format(count($trainSamples)) . "\n";
echo 'Обработано данных для тестирования: ' . number_format(count($testSamples)) . "\n\n";
echo 'Точность: ' . round($score * 100, 2) . '%';
```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Точность: 98.58%
```

**Объяснение:**

Модель выбирает класс, для которого изображение наиболее вероятно. Такая высокая точность объясняется тем, что задача упрощена (только цифры 0 и 1). Для полной задачи с 10 классами качество Gaussian Naive Bayes обычно значительно ниже.

**Что здесь происходит интуитивно**

Каждый пиксель задает вопрос:

> "Насколько это значение похоже на то, что я обычно вижу у этой цифры?"

Каждый пиксель вносит вклад в итоговую вероятность класса. Класс с наибольшей суммарной вероятностью (в логарифмах) побеждает.

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
use app\classes\MnistLoader;
use Rubix\ML\Classifiers\GaussianNB;
use Rubix\ML\CrossValidation\Metrics\Accuracy;
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Extractors\CSV;

// Build the training and test datasets from the filtered CSV rows.
$trainRows = MnistLoader::loadIterable('train.csv', categoricalLabels: true, normalize: true, digits: [0, 1]);
$testRows = MnistLoader::loadIterable('test.csv', categoricalLabels: true, normalize: true, digits: [0, 1]);

$dataset = Labeled::fromIterator($trainRows);
$testDataset = Labeled::fromIterator($testRows);

$model = new GaussianNB();
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

Точность: 99.15%
```

**Объяснение:**

RubixML делает то же самое:

* считает средние и дисперсии по каждому пикселю
* применяет Gaussian Naive Bayes
* возвращает наиболее вероятный класс

Здесь также как и в предыдущем примере такая высокая точность объясняется тем, что задача была упрощена до цифр 0 и 1.

#### Вывод и следующие шаги

Этот кейс завершает линию развития:

* от простых признаков → к тексту → к числовым данным → к изображениям
* от бинарной классификации → к многоклассовой

Наивный Байес показывает здесь свою главную силу:

> он масштабируется по количеству признаков почти без усложнения логики.

Несмотря на сильное предположение о независимости пикселей, модель все равно способна различать цифры – потому что даже приближенные вероятности дают достаточно информации для правильного выбора.

И это один из самых важных уроков машинного обучения: идеальная модель не всегда нужна – достаточно той, которая правильно ранжирует варианты.

Наивный Байес даёт быстрый, но довольно грубый результат из-за сильного предположения независимости признаков. Чтобы учесть реальную структуру данных, в следующей главе мы попробуем более гибкий подход – k ближайших соседей (k-NN), который опирается на схожесть изображений и позволяет моделировать нелинейные границы.

**Прогресс**

Мы будем возвращаться к этому кейсу в следующих главах и постепенно улучшать модель, добавляя новые идеи и методы. Важно не просто увидеть разные алгоритмы, а понять, как они меняют поведение модели на одной и той же задаче. Ниже – прогресс, который мы имеем на текущий момент.

Прогресс моделей на MNIST:

<table><thead><tr><th width="193.01953125">Модель</th><th width="181.01171875">Задача</th><th width="168.92578125">Точность</th><th>Комментарий</th></tr></thead><tbody><tr><td><a href="../../14.-logisticheskaya-regressiya/skvoznoi-keis-raspoznavanie-cifr-mnist/mnist-binarnaya-klassifikaciya-otlichaem-0-ot-1.md">Logistic</a></td><td>0 vs 1 (binary)</td><td>~99%</td><td>линейная</td></tr><tr><td><a href="mnist-veroyatnostnaya-model-klassifikacii-cifr-naive-bayes.md">Naive Bayes</a></td><td>0 vs 1 (binary)</td><td>~98–99%</td><td>простая вероятностная модель, предполагает независимость пикселей</td></tr><tr><td>...</td><td></td><td>...</td><td>далее: k-NN</td></tr></tbody></table>

{% hint style="info" %}
Чтобы самостоятельно протестировать этот код, воспользуйтесь [онлайн-демонстрацией](https://aiwithphp.org/books/ai-for-php-developers/examples/part-3/why-naive-bayes-works) для его запуска.
{% endhint %}
