# Кейс 1. Учебное дерево решений на чистом PHP

В этом кейсе мы разберём дерево решений максимально приземлённо – без библиотек, без оптимизаций и без попыток написать универсальный алгоритм. Цель здесь учебная: увидеть, как идеи энтропии и information gain превращаются в конкретные шаги кода и последовательные решения.

Этот кейс стоит читать не как рецепт для продакшена, а как разбор механики. Если вы поймёте его целиком, любые реализации decision tree в библиотеках перестанут быть чем-то удивительным или магическим.

#### Цель кейса

Цель этого кейса – показать, как дерево решений:

* измеряет неопределённость данных
* выбирает лучший вопрос к данным
* шаг за шагом уточняет решение

Мы сознательно работаем с маленьким набором данных и простыми условиями. Это позволяет сосредоточиться на логике алгоритма, а не на инженерных деталях.

#### Сценарий задачи

Представим простую бизнес-задачу. У нас есть пользователи сайта, и мы хотим разделить их на два класса: "активный" и "пассивный". Для каждого пользователя известны два числовых признака:

* количество визитов в неделю
* среднее время на сайте в минутах

Интуитивно понятно, что активные пользователи чаще заходят на сайт и проводят там больше времени. Дерево решений должно формализовать эту интуицию в виде последовательности вопросов.

#### Исходные данные

Каждый объект описывается тройкой значений: \[visits, time, label]. Данные намеренно сделаны маленькими и наглядными.

```php
$data = [
    [5, 10, 'active'],
    [7, 15, 'active'],
    [1, 2, 'passive'],
    [2, 3, 'passive'],
    [6, 8, 'active'],
    [3, 4, 'passive'],
];
```

Даже визуально видно, что данные уже частично разделимы. В данном примере признаки частично коррелированы между собой, поэтому дерево может выбрать любой из них для первого разбиения – это нормальное поведение алгоритма. Задача дерева – найти такие пороги по признакам, которые сделают это разделение формальным.

#### Энтропия как мера неопределённости

Первое, что нужно дереву решений, – уметь измерять, насколько "чистыми" являются данные в узле. Для этого используется энтропия.

```php
function entropy(array $labels): float {
    $counts = array_count_values($labels);
    $total = count($labels);
    $entropy = 0.0;
    
    if ($total) {
        foreach ($counts as $count) {
            $p = $count / $total;
            $entropy -= $p * log($p, 2);
        }
    }
    
    return $entropy;
}
```

Если в узле все пользователи относятся к одному классу, энтропия равна нулю – неопределённости нет. Если классы перемешаны, энтропия растёт. Это ровно та интуиция, которая нужна дереву для выбора вопросов.

{% hint style="info" %}
В вышеуказанном примере в логарифме используется основание 2, что является стандартом в теории информации. На практике основание логарифма не влияет на выбор разбиений — оно лишь масштабирует значение энтропии.
{% endhint %}

#### Information gain – польза от вопроса

Теперь мы можем оценить, насколько полезен конкретный вопрос к данным. Для этого используется information gain – уменьшение энтропии после разбиения.

```php
function informationGain(array $parent, array $left, array $right): float {
    $total = count($parent);
    
    if (!$total) {
        return 0.0;
    }
    
    $parentLabels = array_column($parent, 2);
    $leftLabels   = array_column($left, 2);
    $rightLabels  = array_column($right, 2);

    $hParent = entropy($parentLabels);
    $hLeft   = entropy($leftLabels);
    $hRight  = entropy($rightLabels);

    $weighted = (count($left) / $total) * $hLeft
              + (count($right) / $total) * $hRight;

    return $hParent - $weighted;
}
```

Если information gain равен нулю, вопрос не уменьшает неопределённость данных и, с точки зрения критерия энтропии, не улучшает разбиение. Чем больше это значение, тем лучше вопрос.

#### Разбиение по признаку

Для простоты мы будем рассматривать только бинарные разбиения вида "признак < порог".&#x20;

В этом учебном примере мы не фиксируем конкретный способ выбора порогов. На практике их обычно берут между соседними уникальными значениями признака и перебирают все такие варианты. Это самый распространённый вариант для числовых признаков.&#x20;

```php
function split(array $data, int $featureIndex, float $threshold): array {
    $left = [];
    $right = [];

    foreach ($data as $row) {
        if ($row[$featureIndex] < $threshold) {
            $left[] = $row;
        } else {
            $right[] = $row;
        }
    }

    return [$left, $right];
}
```

Перебирая возможные пороги и вычисляя information gain, дерево фактически перебирает возможные вопросы и выбирает самый информативный.

#### Пример использования кода

Теперь посмотрим, как этот код применяется на практике.\
Наша цель – найти лучший первый вопрос дерева решений, то есть такое разбиение данных, которое максимальнее всего уменьшает неопределённость.

Для простоты:

* возьмём один признак – `visits`
* переберём возможные пороги
* выберем разбиение с максимальным _information gain_

**Шаг 1. Выбор признака и возможных порогов**

Мы будем использовать признак `visits` (индекс `0`).\
В качестве кандидатов на пороги возьмём все уникальные значения этого признака.

```php
$featureIndex = 0; // visits

$values = array_unique(array_column($data, $featureIndex));
sort($values);
```

Идея простая: дерево перебирает возможные вопросы вида "**visits < threshold?"** и смотрит, какой из них лучше всего упорядочивает данные.

**Шаг 2. Перебор разбиений и подсчёт information gain**

Теперь для каждого порога:

1. разбиваем данные на две части
2. считаем information gain
3. запоминаем лучшее разбиение

```php
$bestGain = 0.0;
$bestThreshold = null;
$bestSplit = null;

foreach ($values as $threshold) {
    [$left, $right] = split($data, $featureIndex, $threshold);

    // Пропускаем вырожденные разбиения
    if (empty($left) || empty($right)) {
        continue;
    }

    $gain = informationGain($data, $left, $right);

    if ($gain > $bestGain) {
        $bestGain = $gain;
        $bestThreshold = $threshold;
        $bestSplit = [$left, $right];
    }
}
```

На этом этапе уже хорошо видно ключевую идею decision tree: алгоритм не подбирает непрерывные коэффициенты и не использует градиентную оптимизацию, а систематически перебирает возможные вопросы к данным и оценивает их пользу.

**Шаг 3. Результат — первый вопрос дерева**

Выведем результат:

```php
echo "Лучшее разбиение:" . PHP_EOL;
echo "Признак: visits" . PHP_EOL;
echo "Порог: $bestThreshold" . PHP_EOL;
echo "Information Gain: $bestGain" . PHP_EOL . PHP_EOL;

echo "Левая ветка:" . PHP_EOL;
echo array_to_matrix($bestSplit[0]) . PHP_EOL;

echo PHP_EOL . "Правая ветка:" . PHP_EOL;
echo array_to_matrix($bestSplit[1]);

// Результат
// Лучшее разбиение:
// Признак: visits
// Порог: 5
// Information Gain: 1

// Левая ветка:
// [1, 2, passive]
// [2, 3, passive]
// [3, 4, passive]

// Правая ветка:
// [5, 10, active]
// [7, 15, active]
// [6, 8, active]
```

Интерпретация результата будет выглядеть так:

> Если visits < threshold — идём в левую ветку, иначе в правую.

Это и есть **первый узел дерева решений**.

<details>

<summary>Кейс 1. Полный пример кода на чистом PHP</summary>

```php
$data = [
    [5, 10, 'active'],
    [7, 15, 'active'],
    [1, 2, 'passive'],
    [2, 3, 'passive'],
    [6, 8, 'active'],
    [3, 4, 'passive'],
];

function entropy(array $labels): float {
    $counts = array_count_values($labels);
    $total = count($labels);

    $entropy = 0.0;
    foreach ($counts as $count) {
        $p = $count / $total;
        $entropy -= $p * log($p, 2);
    }

    return $entropy;
}

function informationGain(array $parent, array $left, array $right): float {
    $parentLabels = array_column($parent, 2);
    $leftLabels   = array_column($left, 2);
    $rightLabels  = array_column($right, 2);

    $hParent = entropy($parentLabels);
    $hLeft   = entropy($leftLabels);
    $hRight  = entropy($rightLabels);

    $total = count($parent);

    $weighted = (count($left) / $total) * $hLeft
              + (count($right) / $total) * $hRight;

    return $hParent - $weighted;
}

function split(array $data, int $featureIndex, float $threshold): array {
    $left = [];
    $right = [];

    foreach ($data as $row) {
        if ($row[$featureIndex] < $threshold) {
            $left[] = $row;
        } else {
            $right[] = $row;
        }
    }

    return [$left, $right];
}

// Возможные признаки: 0 = visits, 1 = time
$featureIndex = 0;

// Возьмём все уникальные значения признака как кандидаты на пороги
$values = array_unique(array_column($data, $featureIndex));
sort($values);

$bestGain = 0.0;
$bestThreshold = null;
$bestSplit = null;

foreach ($values as $threshold) {
    [$left, $right] = split($data, $featureIndex, $threshold);

    // пропускаем вырожденные разбиения
    if (empty($left) || empty($right)) {
        continue;
    }

    $gain = informationGain($data, $left, $right);

    if ($gain > $bestGain) {
        $bestGain = $gain;
        $bestThreshold = $threshold;
        $bestSplit = [$left, $right];
    }
}

echo "Лучшее разбиение:" . PHP_EOL;
echo "Признак: visits" . PHP_EOL;
echo "Порог: $bestThreshold" . PHP_EOL;
echo "Information Gain: $bestGain" . PHP_EOL . PHP_EOL;

echo "Левая ветка:" . PHP_EOL;
echo array_to_matrix($bestSplit[0]) . PHP_EOL;

echo PHP_EOL . "Правая ветка:" . PHP_EOL;
echo array_to_matrix($bestSplit[1]);
```

</details>

#### Как из этого вырастает дерево

В полном алгоритме этот процесс выполняется рекурсивно. После выбора лучшего разбиения мы повторяем тот же самый шаг для каждого дочернего узла, пока:

* данные не станут полностью однородными
* или не будет достигнута максимальная глубина
* или в узле не останется слишком мало объектов

Даже без полной рекурсивной реализации уже видно, как формируется структура дерева и почему оно выглядит именно так, а не иначе. В библиотечных реализациях всё это автоматизировано и оптимизировано, но логика остаётся ровно такой же, как в этом примере.

#### Что важно понять из кейса

Этот кейс не про скорость и не про масштабируемость. Он про понимание.

Если свести его к сути, дерево решений:

* измеряет неопределённость
* выбирает вопрос, который уменьшает эту неопределённость
* повторяет процесс, пока это имеет смысл

Если вы можете мысленно пройти этот путь – от исходных данных до выбранного порога – значит вы понимаете decision tree на уровне механики. Всё остальное, включая оптимизации и библиотеки, является лишь надстройкой над этой базовой логикой.
