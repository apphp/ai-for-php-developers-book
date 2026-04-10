# MNIST: бинарная классификация - отличаем 0 от 1

Это один из самых известных учебных кейсов в машинном обучении.

Он простой по постановке, но при этом очень наглядно показывает, как работает модель – и где она начинает ломаться.

Здесь появляется важный "поворот" в нашей книге: мы переходим от табличных данных к изображениям.

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

Важно проговорить для себя и запомнить:

> Изображение – это просто массив чисел.

Нет "форм", "контуров" или "линий" – только значения пикселей.

И модель учится находить закономерности в этих числах.

#### Модель

В этой главе мы рассматриваем логистическую регрессию:

$$
z = w \cdot x + b
$$

затем

$$
p = \frac{1}{1 + e^{-z}}
$$

где:

* $$x$$ – 784 пикселя
* $$w$$ – 784 веса

Если:

* $$p \approx 1$$ → это "1"
* $$p \approx 0$$ → это "0"

### Подготовка данных

Мы используем реальный MNIST датасет (в формате CSV):

* первый столбец – label
* остальные 784 – пиксели

Что делаем:

* оставляем только 0 и 1
* нормализуем пиксели: делим на 255
* делим на две подвыборки (subsets): train и test

{% hint style="info" %}
Для практических примеров мы будем использовать датасет MNIST в формате CSV.

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

Каждая строка:

```
label,1x1,1x2,1x3,1x4,1x5,1x6,...,,1x26,1x27,1x28,2x1,2x2,2x3,2x4,2x5,2x6,..,28x28
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

<summary><strong>LogisticRegression.php</strong></summary>

```php
class LogisticRegression
{
    private array $weights;
    private float $bias;
    private float $learningRate;

    public function __construct(int $numFeatures, float $learningRate = 0.1)
    {
        $this->learningRate = $learningRate;
        $this->weights = array_fill(0, $numFeatures, 0.0);
        $this->bias = 0.0;
    }

    private function sigmoid(float $z): float
    {
        return 1.0 / (1.0 + exp(-$z));
    }

    private function dot(array $a, array $b): float
    {
        $sum = 0.0;
        foreach ($a as $i => $v) {
            $sum += $v * $b[$i];
        }
        return $sum;
    }

    public function predictProb(array $x): float
    {
        return $this->sigmoid($this->dot($this->weights, $x) + $this->bias);
    }

    public function predict(array $x): int
    {
        return $this->predictProb($x) >= 0.5 ? 1 : 0;
    }

    public function train(array $X, array $y, int $epochs = 5): void
    {
        foreach (range(1, $epochs) as $epoch) {
            foreach ($X as $i => $x) {
                $p = $this->predictProb($x);
                $error = $p - $y[$i];

                foreach ($this->weights as $j => $w) {
                    $this->weights[$j] -= $this->learningRate * $error * $x[$j];
                }

                $this->bias -= $this->learningRate * $error;
            }

            echo "Epoch $epoch done\n";
        }
    }
}
```



</details>

Теперь загрузим две наши подвыборки&#x20;

```php
function loadMNIST(string $file): array {
    $X = [];
    $y = [];

    $handle = fopen($file, 'r');

    while (($row = fgetcsv($handle)) !== false) {

        $label = (int)$row[0];

        // 1. Оставляем только 0 и 1
        if ($label !== 0 && $label !== 1) {
            continue;
        }

        $pixels = array_slice($row, 1);
        
        // 2. Нормализация (0–255 → 0–1)
        $pixels = array_map(fn($p) => $p / 255.0, $pixels);

        $X[] = $pixels;
        $y[] = $label;
    }

    return [$X, $y];
}

[$X_train, $y_train] = loadMNIST('mnist/train.csv');
[$X_test, $y_test] = loadMNIST('mnist/test.csv');
```

И начнём и тренировать:

```php
$model = new LogisticRegression(784, 0.1);

$model->train($X_train, $y_train, 3);

// оценка
$correct = 0;

foreach ($X_test as $i => $x) {
    if ($model->predict($x) === $y_test[$i]) {
        $correct++;
    }
}

$accuracy = $correct / count($X_test);

echo "Accuracy: " . round($accuracy * 100, 2) . "%\n";
```

Результат:&#x20;

Accuracy: 95.21%

Объяснение:

Для данной задачи 0 vs 1 MNIST (очень простой) – классы хорошо разделимы, поэтому даже линейная модель даёт \~95% точности.

### Реализация с RubixML

```php
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Classifiers\LogisticRegression;

function loadMNIST(string $file): Labeled
{
    $samples = [];
    $labels = [];

    $handle = fopen($file, 'r');

    while (($row = fgetcsv($handle)) !== false) {

        $label = (int)$row[0];

        // 1. Оставляем только 0 и 1
        if ($label !== 0 && $label !== 1) {
            continue;
        }

        $pixels = array_slice($row, 1);
        
        // 2. Нормализация (0–255 → 0–1)
        $pixels = array_map(fn($p) => $p / 255.0, $pixels);

        $samples[] = $pixels;
        $labels[] = $label;
    }

    return new Labeled($samples, $labels);
}

$train = loadMNIST('train.csv');
$test = loadMNIST('test.csv');

$model = new LogisticRegression(0.1, 100);
$model->train($train);

$predictions = $model->predict($test);

$correct = 0;

foreach ($predictions as $i => $pred) {
    if ($pred == $test->labels()[$i]) {
        $correct++;
    }
}

echo "Accuracy: " . round($correct / count($predictions) * 100, 2) . "%\n";
```

Результат:&#x20;

Accuracy: 98.45%

Объяснение:

Для данной задачи 0 vs 1 MNIST (очень простой) – классы хорошо разделимы, но RubixML может дать чуть лучше за счёт более стабильной реализации и оптимизаций

#### Где модель ломается

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

Но у неё есть фундаментальное ограничение.

#### Выводы и следующие шаги

Этот кейс – важная точка в понимании машинного обучения. Он наглядно показывает, как изображения превращаются в числа, как модели работают в пространствах высокой размерности и что логистическая регрессия в целом хорошо масштабируется даже на задачи с сотнями признаков.

Однако ключевое ограничение становится очевидным: модель способна построить только линейную границу. В случае с MNIST это означает попытку разделить пространство из 784 признаков одной гиперплоскостью. И хотя результат оказывается лучше, чем можно было бы ожидать, модель всё равно упирается в геометрию данных – рукописные цифры по своей природе не являются линейно разделимыми структурами.

Отсюда возникает естественный вопрос:&#x20;

> А можно ли подойти к задаче иначе – не через поиск границы между классами, а через моделирование вероятности самих данных?

К этому подходу мы перейдём в следующей главе, где рассмотрим вероятностную модель классификации – Naive Bayes.
