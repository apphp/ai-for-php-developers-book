---
description: Использование косинусного сходства
---

# Кейс 3: Сравнение текстов

Предположим, мы представили тексты в виде векторов – через bag-of-words, TF-IDF или эмбеддинги языковой модели. Каждый текст теперь – точка в пространстве.

<div align="left"><figure><img src="../../../.gitbook/assets/7.5-texts-as-points-in-vector-space (1).png" alt="" width="563"><figcaption><p>Тексты как точки в векторном пространстве</p></figcaption></figure></div>

Как мы уже обсуждали, евклидово расстояние плохо подходит для сравнения текстов разной длины. Косинусное сходство, напротив, учитывает только направление векторов, то есть относительное распределение слов и смыслов. Поэтому оно широко используется в задачах поиска по документам и семантического сравнения.

Алгоритм при этом очень простой:

1. Преобразовать текст в вектор.
2. Посчитать косинусное сходство с другими.
3. Отсортировать по убыванию сходства.

#### Вариант 1. Чистый PHP (TF-IDF + cosine similarity)

**Простейшая векторизация (TF)**

Для примера – без лемматизации и стоп-слов.

```php
function textToVector(string $text): array {
    $tokens = preg_split('/\W+/u', mb_strtolower($text), -1, PREG_SPLIT_NO_EMPTY);
    $vector = [];

    foreach ($tokens as $token) {
        $vector[$token] = ($vector[$token] ?? 0) + 1;
    }

    return $vector;
}
```

**Приведение векторов к общему пространству**

(один и тот же набор слов → одинаковый порядок координат)

```php
function alignVectors(array $a, array $b): array {
    $keys = array_unique(array_merge(array_keys($a), array_keys($b)));

    $va = [];
    $vb = [];

    foreach ($keys as $key) {
        $va[] = $a[$key] ?? 0;
        $vb[] = $b[$key] ?? 0;
    }

    return [$va, $vb];
}
```

**Косинусное сходство (логика)**

```php
function dotProduct(array $a, array $b): float {
    $n = count($a);

    if ($n !== count($b)) {
        throw new InvalidArgumentException('Vectors must have the same length');
    }

    $sum = 0.0;

    for ($i = 0; $i < $n; $i++) {
        $sum += $a[$i] * $b[$i];
    }

    return $sum;
}
```

```php
function cosineSimilarity(array $a, array $b): float {
    $dot = dotProduct($a, $b);
    $normA = sqrt(dotProduct($a, $a));
    $normB = sqrt(dotProduct($b, $b));

    if ($normA == 0.0 || $normB == 0.0) {
        return 0.0;
    }

    return $dot / ($normA * $normB);
}
```

**Пример поиска по документам**

```php
$documents = [
    'Машинное обучение используется для анализа данных, который очень важен для результата',
    'Глубинное обучение – раздел машинного обучения',
    'Кошки и собаки – популярные домашние животные',
];

$query = 'алгоритмы машинного обучения';

$queryVector = textToVector($query);

$scores = [];

foreach ($documents as $doc) {
    $docVector = textToVector($doc);

    [$v1, $v2] = alignVectors($queryVector, $docVector);
    $score = cosineSimilarity($v1, $v2);

    $scores[] = [
        'text' => $doc,
        'score' => $score,
    ];
}

usort($scores, fn($a, $b) => $b['score'] <=> $a['score']);

print_r($scores);
```

Результат:

```
Array (
    [0] => Array (
        [text] => Глубинное обучение – раздел машинного обучения
        [score] => 0.51639777949432
    )
    [1] => Array (
        [text] => Машинное обучение используется для анализа данных, который очень важен для результата
        [score] => 0
    )
    [2] => Array (
        [text] => Кошки и собаки – популярные домашние животные
        [score] => 0
    )
)
```

**Что делает алгоритм**

1. Преобразуем текст в вектор
2. Считаем cosine similarity
3. Сортируем документы по убыванию сходства

{% hint style="info" %}
В данном примере тексты про машинное обучение второго документа оказываются ближе к запросу, независимо от их длины, потому что именно они имеют значимые компоненты в тех же координатах TF-IDF-пространства, а косинусное сходство не учитывает длину документа.
{% endhint %}

Небольшой дебаг для большего понимания процесса:

```
// Результат: textToVector($queryVector)
[1, 1, 1]

// Результаты: alignVectors($queryVector, $docVector)
// Первый документ:
tensor([
   [1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
   [0, 0, 0, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1]
])
// Второй документ:
tensor([
   [1, 1, 1, 0, 0, 0],
   [0, 1, 1, 1, 1, 1]
])
// Третий документ:
tensor([
   [1, 1, 1, 0, 0, 0, 0, 0, 0],
   [0, 0, 0, 1, 1, 1, 1, 1, 1]
])
```

#### Вариант 2. Реализация с использованием Rubix ML

**Пример: поиск похожих документов**

```php
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Transformers\TextNormalizer;
use Rubix\ML\Transformers\WordCountVectorizer;
use Rubix\ML\Transformers\TfIdfTransformer;
use Rubix\ML\Kernels\Distance\Cosine;

// Набор документов
$documents = [
    'Машинное обучение используется для анализа данных, который очень важен для результата',
    'Глубинное обучение — раздел машинного обучения',
    'Кошки и собаки — популярные домашние животные',
];

// Поисковый запрос
$query = ['алгоритмы машинного обучения'];

// Объединяем документы и запрос в один датасет
$dataset = new Unlabeled(array_merge($documents, $query));

// Нормализация текста и векторизация слов
$dataset->apply(new TextNormalizer());
$dataset->apply(new WordCountVectorizer());

// Преобразуем частоты слов в TF-IDF векторы
$tfidf = new TfIdfTransformer();
$tfidf->fit($dataset);
$dataset->apply($tfidf);

// Получаем числовые векторы
$vectors = $dataset->samples();

// Последний вектор – это запрос
$queryVector = array_pop($vectors);

// Косинусная метрика
$kernel = new Cosine();

$scores = [];

foreach ($vectors as $i => $vector) {
    // Rubix ML возвращает расстояние, поэтому используем 1 - distance
    $similarity = 1.0 - $kernel->compute($queryVector, $vector);

    $scores[] = [
        'text' => $documents[$i],
        'score' => $similarity,
    ];
}

// Сортируем документы по убыванию сходства
usort($scores, fn($a, $b) => $b['score'] <=> $a['score']);

print_r($scores);
```

Результат:

```
Array (
    [0] => Array (
        [text] => Глубинное обучение – раздел машинного обучения
        [score] => 0.4222221617033
    )
    [1] => Array (
        [text] => Машинное обучение используется для анализа данных, который очень важен для результата
        [score] => 0
    )
    [2] => Array (
        [text] => Кошки и собаки — популярные домашние животные
        [score] => 0
    )
)
```

**Что здесь происходит**

1. **Преобразование текста в векторы**\
   С помощью `TfIdfTransformer` каждый текст переводится в числовое представление.
2. **Сравнение с запросом**\
   Для каждого документа считается косинусное сходство с вектором запроса.
3. **Сортировка результатов**\
   Документы упорядочиваются по убыванию сходства – наиболее релевантные оказываются первыми.

#### Вывод

Даже если документы имеют разную длину, косинусное сходство позволяет корректно сравнивать их по смыслу. Именно поэтому такой подход лежит в основе большинства систем полнотекстового и семантического поиска.
