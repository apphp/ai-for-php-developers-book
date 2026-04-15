# MNIST: бинарная классификация - отличаем 0 от 1

Это один из самых известных учебных кейсов в машинном обучении.

Он прост в установке и запуске, но при этом очень наглядно показывает, как работает модель – и где проявляются её ограничения.

Кроме того, мы с вами совершаем важный "поворот" в нашей книге, а именно: мы переходим от табличных данных к изображениям.

#### Цель кейса

Научиться классифицировать изображения цифр как "0" или "1" с помощью логистической регрессии и понять ограничения линейной модели.

#### Сценарий

У нас есть датасет MNIST – набор изображений рукописных цифр.

Каждое изображение:

* размер: 28×28 пикселей
* всего: 784 пикселя

Модель не "видит" цифру как человек.

Она получает:

$$
x = [p_1, p_2, ..., p_{784}]
$$

где каждый $$p_i$$ — интенсивность пикселя (0–255 или 0–1 после нормализации).

Задача:

* если цифра = 0 → класс 0
* если цифра = 1 → класс 1

Это бинарная классификация, но уже в 784-мерном пространстве.

#### Как модель "видит" изображение

Важно проговорить про себя и запомнить, чтобы была ясность:

> Изображение – это просто массив чисел.

Модель не имеет явного представления о "формах", "контурах" или "линиях" – она работает только с числовыми значениями пикселей. И она учится находить закономерности в этих числах.

#### Модель

В этой главе мы рассматриваем логистическую регрессию, где применяется:

$$
z = w \cdot x + b
$$

и

$$
p = \frac{1}{1 + e^{-z}}
$$

При этом:

* $$x$$ – 784 пикселя
* $$w$$ – 784 веса

Если:

* $$p \approx 1$$ → это "1"
* $$p \approx 0$$ → это "0"

#### Подготовка данных

Мы используем реальный MNIST датасет (в формате CSV - считайте, что это картинка, которая уже была трансформирована в удобный нам формат):

* первый столбец – label
* остальные 784 – пиксели

Что мы делаем:

* оставляем только 0 и 1
* нормализуем пиксели: делим значение пикселя на 255
* затем нужно разделить датасет на две подвыборки (subsets): train и test, но в нашем случае мы используем уже готовое разбиение MNIST на train и test, поэтому дополнительное разделение не требуется

{% hint style="info" %}
Использование датасет MNIST в формате CSV для практических примеров.

1. Перейдите на страницу:\
   [https://www.kaggle.com/datasets/oddrationale/mnist-in-csv](https://www.kaggle.com/datasets/oddrationale/mnist-in-csv)
2. Скачайте файлы:
   * mnist\_train.csv
   * mnist\_test.csv
3. Поместите их в папку своего проекта:

```
mnist/
 ├── train.csv
 └── test.csv
```

\
CSV-формат позволяет работать с данными напрямую в PHP без дополнительных библиотек.

Что внутри:

Каждая строка хранится в следующем формате:

```
label,1x1,1x2,1x3,1x4,1x5,1x6,...,1x26,1x27,1x28,2x1,2x2,2x3,2x4,2x5,2x6,..,28x28
```

Пример:

```
7,0,0,0,0,0,0,0,0,169,253,253,253,253,253,253,218,30,0,0,0,0,0,0,0,0,0,0,0,0,...,0
```
{% endhint %}

#### Визуализация

<div align="left"><figure><img src="../../../.gitbook/assets/14.4-mnist-samples-0-1.png" alt="" width="563"><figcaption><p>14.4 MNIST примеры</p></figcaption></figure></div>

#### Реализация на чистом PHP

Для начала создадим класс логистической регрессии.&#x20;

<details>

<summary><strong>Class LogisticRegression</strong></summary>

```php
class LogisticRegression {
    private array $weights;
    private float $bias;
    private float $learningRate;

    public function __construct(int $numFeatures, float $learningRate = 0.1) {
        // Start with zeroed parameters for every feature.
        $this->learningRate = $learningRate;
        $this->weights = array_fill(0, $numFeatures, 0.0);
        $this->bias = 0.0;
    }

    // Convert a linear score into a probability in the range [0, 1].
    private function sigmoid(float $z): float {
        return 1.0 / (1.0 + exp(-$z));
    }

    // Compute the weighted sum of the input features.
    private function dot(array $a, array $b): float {
        $sum = 0.0;
        foreach ($a as $i => $v) {
            $sum += $v * $b[$i];
        }
        return $sum;
    }

    // Return the probability that the sample belongs to class 1.
    public function predictProb(array $x): float {
        return $this->sigmoid($this->dot($this->weights, $x) + $this->bias);
    }

    // Convert the probability into a binary class prediction.
    public function predict(array $x): int {
        return $this->predictProb($x) >= 0.5 ? 1 : 0;
    }

    // Train the model with simple gradient descent.
    public function train(array $X, array $y, int $epochs = 5): void {
        foreach (range(1, $epochs) as $epoch) {
            foreach ($X as $i => $x) {
                // Compare the prediction to the expected label.
                $p = $this->predictProb($x);
                $error = $p - $y[$i];

                // Update each weight using the feature value and error.
                foreach ($this->weights as $j => $w) {
                    $this->weights[$j] -= $this->learningRate * $error * $x[$j];
                }

                // Update the bias term separately.
                $this->bias -= $this->learningRate * $error;
            }

            // echo "Epoch $epoch done\n";
        }
    }

    // Calculate model accuracy
    public function accuracy(array $X, array $y): float {
        $correct = 0;

        foreach ($X as $i => $x) {
            if ($this->predict($x) === $y[$i]) {
                $correct++;
            }
        }

        return count($X) > 0 ? ($correct / count($X)) : 0.0;
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
[$X_train, $y_train] = MnistLoader::loadMNIST('train.csv');
[$X_test, $y_test] = MnistLoader::loadMNIST('test.csv');

$model = new LogisticRegression(784, 0.1);
$model->train($X_train, $y_train, epochs: $epochs = 5);

echo 'Number of epochs: ' . $epochs . "\n";

// Calculate model accuracy
$accuracy = $model->accuracy($X_test, $y_test);

echo 'Train samples handled: ' . number_format(count($X_train)) . "\n";
echo 'Test samples handled: ' . number_format(count($X_test)) . "\n\n";
echo 'Accuracy: ' . round($accuracy * 100, 2) . '%';

```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Количество эпох: 5

Точность: 99.91%
```

**Объяснение:**

Для данной задачи с MNIST 0 vs 1 (относительно простой) – классы почти линейно разделимы в пространстве признаков, поэтому даже простая линейная модель даёт \~99% точности.

#### Реализация с RubixML

```php
use Rubix\ML\Classifiers\LogisticRegression;
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Extractors\CSV;

function mnistRows(string $file): iterable {
    foreach (new CSV($file, false) as $row) {
        if (!isset($row[0])) {
            continue;
        }

        $label = (int) $row[0];

        // 1. Оставляем только 0 и 1
        if ($label !== 0 && $label !== 1) {
            continue;
        }

        // 2. Нормализация (0–255 → 0–1)
        $pixels = array_map(static fn ($value): float => ((float) $value) / 255.0, array_slice($row, 1));
        
        yield array_merge($pixels, [$label === 1 ? 'one' : 'zero']);
    }
}

$trainRows = mnistRows('train.csv');
$testRows = mnistRows('test.csv');

$dataset = Labeled::fromIterator($trainRows);
$testDataset = Labeled::fromIterator($testRows);

$model = new LogisticRegression(epochs: 5);
$model->train($dataset);

$correct = 0;

foreach ($testDataset->samples() as $i => $x) {
    $prediction = $model->predict(new Unlabeled([$x]))[0];

    if ($prediction === $testDataset->labels()[$i]) {
        $correct++;
    }
}

$accuracy = $correct / $testDataset->numSamples();

echo 'Обработано данных для обучения: ' . $dataset->numSamples() . "\n";
echo 'Обработано данных для тестирования: ' . $testDataset->numSamples() . "\n\n";
echo 'Количество эпох: ' . $model->params()['epochs'] . "\n\n";
echo 'Точность: ' . round($accuracy * 100, 2) . '%';
```

**Результат:**

```
Обработано данных для обучения: 12,666
Обработано данных для тестирования: 2,116

Количество эпох: 5

Точность: 99.95%
```

**Объяснение:**

Не смотря на то, что для данной задачи 0 vs 1  – классы хорошо разделимы, тем не менее RubixML может дать результат чуть лучше (99.95%) за счёт отличий в реализации оптимизации (например, регуляризации и более стабильных численных методов).

#### Где модель ломается (проявляются её ограничения)

Это ключевой момент кейса.

Логистическая регрессия может провести только одну гиперплоскость.

Но изображения цифр:

* имеют сложную форму
* сильно варьируются
* иногда выглядят неоднозначно

Примеры проблем:

* "кривые" единицы похожи на нули
* "тонкие" нули похожи на единицы
* шум и артефакты

<div align="left"><figure><img src="../../../.gitbook/assets/14.5-mnist-errors-0-1.png" alt="" width="563"><figcaption><p>14.5 Ошибки классификации MNIST</p></figcaption></figure></div>

#### Практический результат

Даже при всей простоте модель показывает неожиданно хороший результат.

Для нас важно подчеркнуть:

> Линейная модель уже умеет решать нетривиальные задачи.

Но важно понимать, что такой результат – скорее исключение, чем правило, и у модели есть фундаментальные ограничения (см. выше)

#### Выводы и следующие шаги

Этот кейс – важная точка в понимании машинного обучения. Он наглядно показывает, как изображения превращаются в числа, как модели работают в пространствах высокой размерности и что логистическая регрессия в целом хорошо масштабируется даже на задачи с сотнями признаков.

Однако ключевое ограничение становится очевидным: модель способна построить только линейную границу. В случае с MNIST это означает попытку разделить пространство из 784 признаков одной гиперплоскостью. И хотя результат оказывается лучше, чем можно было бы ожидать, модель всё равно упирается в геометрию данных – рукописные цифры по своей природе не являются линейно разделимыми структурами.

Отсюда возникает естественный вопрос:&#x20;

> А можно ли подойти к задаче иначе – не через поиск границы между классами, а через моделирование вероятности самих данных?

Ответ - да, это возможно. Но к этому подходу мы перейдём в следующей главе, где рассмотрим вероятностную модель классификации – Naive Bayes.

**Прогресс**

Мы будем возвращаться к этому кейсу в разных главах и постепенно улучшать модель – от простой линейной до более мощных методов. Ниже – прогресс, который мы будем строить по ходу книги.

Прогресс моделей на MNIST:

<table><thead><tr><th width="193.01953125">Модель</th><th width="181.01171875">Задача</th><th width="168.92578125">Точность</th><th>Комментарий</th></tr></thead><tbody><tr><td><a href="mnist-binarnaya-klassifikaciya-otlichaem-0-ot-1.md">Logistic</a></td><td>0 vs 1 (binary)</td><td>~99%</td><td>линейная</td></tr><tr><td>...</td><td></td><td>...</td><td>...</td></tr></tbody></table>

{% hint style="info" %}
Чтобы самостоятельно протестировать этот код, воспользуйтесь [онлайн-демонстрацией](https://aiwithphp.org/books/ai-for-php-developers/examples/part-3/logistic-regression) для его запуска.
{% endhint %}
