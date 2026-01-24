---
description: Понять механику
---

# Кейс 1. Категориальные признаки и частоты

## Наивный Байес из частот (чистая логика)

Этот кейс – самый простой из возможных, и именно поэтому он важен. Здесь нет сложных данных, нет числовых распределений и нет "магии обучения". Есть только частоты, условные вероятности и логарифмы – ровно то, о чем шла речь в теоретической главе.

### Наивный Байес без библиотек

#### Цель кейса

Показать, как наивный Байес:

* реализуется напрямую из формулы Байеса
* работает с категориальными признаками
* принимает решение без какого-либо оптимизационного процесса

Этот кейс не про эффективность и не про качество предсказаний. Его задача – связать теорию с кодом буквально построчно.

#### Сценарий

Мы классифицируем пользователя как:

* buyer – пользователь, который с большей вероятностью совершит покупку,
* browser – пользователь, который просто просматривает сайт.

Каждый пользователь описывается двумя бинарными признаками:

* `from_ads` – пришел ли пользователь из рекламы,
* `has_account` – есть ли у него аккаунт.

Признаки простые, но этого достаточно, чтобы увидеть механику модели.

#### Обучающая выборка

Начнем с небольшого набора наблюдений:

```php
$data = [
    ['class' => 'buyer',   'from_ads' => true,  'has_account' => true],
    ['class' => 'buyer',   'from_ads' => false, 'has_account' => true],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
];
```

Здесь важно подчеркнуть: мы не "тренируем" модель в привычном смысле. Мы просто собираем статистику.

#### Подсчет априорных вероятностей классов

Первый шаг – оценка вероятности каждого класса:

```php
$classCounts = [];

foreach ($data as $row) {
    $class = $row['class'];
    $classCounts[$class] = ($classCounts[$class] ?? 0) + 1;
}
```

Фактически это оценка P(C).

Если класс встречается чаще – он априори вероятнее.

#### Подсчет условных вероятностей признаков

Теперь считаем, как часто каждый признак принимает то или иное значение внутри класса:

```php
$featureCounts = [];

foreach ($data as $row) {
    $class = $row['class'];

    foreach ($row as $feature => $value) {
        if ($feature === 'class') {
            continue;
        }

        $featureCounts[$class][$feature][$value] =
            ($featureCounts[$class][$feature][$value] ?? 0) + 1;
    }
}
```

Здесь реализовано ключевое предположение наивного Байеса: каждый признак учитывается независимо от других, но внутри класса.

#### Классификация нового пользователя

Допустим, пришел новый пользователь:

* из рекламы,
* с аккаунтом.

```php
$input = [
    'from_ads' => true,
    'has_account' => true,
];
```

Для каждого класса мы считаем:

$$
P(class | features) ∝ P(class) · Π P(feature | class)
$$

На практике – в логарифмах:

```php
$scores = [];

$totalSamples = count($data);

foreach ($classCounts as $class => $count) {
    $logProb = log($count / $totalSamples);

    foreach ($input as $feature => $value) {
        $featureCount = $featureCounts[$class][$feature][$value] ?? 1;
        $logProb += log($featureCount / $count);
    }

    $scores[$class] = $logProb;
}

arsort($scores);
print_r($scores);

// Результат
// Array (
//     [buyer] => -1.6739764335717
//     [browser] => -2.3671236141316
// )
```

Каждый признак добавляет свой вклад в итоговый логарифм вероятности.

Побеждает класс с наибольшим значением, в нашем случае это `buyer`.

#### Что здесь принципиально важно

В этом коде хорошо видно несколько вещей.

Во-первых, никакого обучения как процесса нет. Нет итераций, нет функции ошибки, нет настройки параметров.

Во-вторых, модель полностью объяснима. В любой момент можно посмотреть, какой признак и как повлиял на решение.

В-третьих, этот код – почти прямая транскрипция формул из предыдущей главы. Если убрать PHP-синтаксис, останется чистая математика.

#### Типичные упрощения и допущения

Здесь намеренно опущены детали, которые важны в реальных задачах:

* сглаживание (Laplace smoothing)
* работа с отсутствующими признаками
* масштабирование данных

Все это не меняет сути модели, а лишь делает ее устойчивее на больших и шумных данных.

<details>

<summary>Кейс 1. Полный пример кода на чистом PHP</summary>

```php
$data = [
    ['class' => 'buyer',   'from_ads' => true,  'has_account' => true],
    ['class' => 'buyer',   'from_ads' => false, 'has_account' => true],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
    ['class' => 'browser', 'from_ads' => true,  'has_account' => false],
];

// Подсчет априорных вероятностей
$classCounts = [];
$featureCounts = [];

foreach ($data as $row) {
    $class = $row['class'];
    $classCounts[$class] = ($classCounts[$class] ?? 0) + 1;

    foreach ($row as $feature => $value) {
        if ($feature === 'class') continue;
        $featureCounts[$class][$feature][$value] =
            ($featureCounts[$class][$feature][$value] ?? 0) + 1;
    }
}

// Классификация нового объекта
$input = ['from_ads' => true, 'has_account' => true];

$scores = [];

foreach ($classCounts as $class => $count) {
    $logProb = log($count / count($data));

    foreach ($input as $feature => $value) {
        $featureCount = $featureCounts[$class][$feature][$value] ?? 1;
        $total = $classCounts[$class];
        $logProb += log($featureCount / $total);
    }

    $scores[$class] = $logProb;
}

arsort($scores);
print_r($scores);
```

</details>

### Та же модель с использованием RubixML

Теперь посмотрим, как та же самая идея выглядит при использовании библиотеки.

```php
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Classifiers\NaiveBayes;

$samples = [
    ['from_ads', 'has_account'],
    ['organic_search', 'has_account'],
    ['from_ads', 'no_account'],
    ['from_ads', 'no_account'],
];

// Метки классов для каждой строки
$labels = ['buyer', 'buyer', 'browser', 'browser'];

// Собираем обучающую выборку.
$dataset = new Labeled($samples, $labels);

// Модель наивного Байеса из RubixML
$model = new NaiveBayes();
$model->train($dataset);

// Классификация нового объекта
$sample = ['from_ads', 'has_account'];

$dataset = new Unlabeled([$sample]);
$prediction = $model->predict($dataset);
print_r($prediction);

// Результат
// Array (
//     [0] => buyer
// )
```

Как видно из резельтатов, в данном случае опять побеждает `buyer`.

RubixML:

* автоматически считает те же частоты
* применяет логарифмы
* выбирает класс с максимальной вероятностью

Разница только в уровне абстракции, а не в логике.

### Вывод

Этот кейс показывает, что наивный Байес – это не "модель из библиотеки", а конкретный алгоритм, который можно реализовать напрямую за несколько десятков строк.

Когда вы используете RubixML, вы не меняете подход – вы просто перестаете писать этот код вручную. Понимание того, что именно библиотека делает за вас, и есть главная ценность этого кейса.

Если после него наивный Байес кажется слишком простым – значит, он наконец стал понятным.
