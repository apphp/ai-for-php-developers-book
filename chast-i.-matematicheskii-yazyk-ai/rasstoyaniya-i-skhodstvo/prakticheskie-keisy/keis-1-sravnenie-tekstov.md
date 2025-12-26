# Кейс 1: сравнение текстов

Предположим, мы представили тексты в виде векторов – через bag-of-words, TF-IDF или эмбеддинги языковой модели. Каждый текст теперь – точка в пространстве.

<div align="left"><figure><img src="../../../.gitbook/assets/7.5-texts-as-points-in-vector-space (1).png" alt="" width="563"><figcaption><p>Тексты как точки в векторном пространстве</p></figcaption></figure></div>

Как мы уже обсуждали, евклидово расстояние плохо подходит для сравнения текстов разной длины. Косинусное сходство, напротив, учитывает только направление векторов, то есть относительное распределение слов и смыслов. Поэтому оно широко используется в задачах поиска по документам и семантического сравнения.

Алгоритм при этом очень простой:

1. Преобразовать текст в вектор.
2. Посчитать косинусное сходство с другими.
3. Отсортировать по убыванию сходства.

### Вариант 1. Чистый PHP (TF-IDF + cosine similarity)

#### Простейшая векторизация (TF)

Для примера — без лемматизации и стоп-слов.

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

#### Приведение векторов к общему пространству

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

#### Косинусное сходство (ваша логика)

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

#### Пример поиска по документам

```php
$documents = [
    'Машинное обучение используется для анализа данных',
    'Глубинное обучение — раздел машинного обучения',
    'Кошки и собаки — популярные домашние животные',
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

......

#### Что делает алгоритм (ровно как в кейсе)

```php
// 1. Преобразуем текст в вектор
// 2. Считаем cosine similarity
// 3. Сортируем документы по убыванию сходства
```

Результат: тексты **про машинное обучение** окажутся выше,\
независимо от их длины.

### Вариант 2. Реализация с использованием Rubix ML.

#### Пример: поиск похожих документов

```php
use Rubix\ML\Datasets\Unlabeled;
use Rubix\ML\Transformers\TfIdfTransformer;
use Rubix\ML\Kernels\Distance\Cosine;

// Набор документов
$documents = [
    'Машинное обучение используется для анализа данных',
    'Глубинное обучение — раздел машинного обучения',
    'Кошки и собаки — популярные домашние животные',
];

// Поисковый запрос
$query = ['алгоритмы машинного обучения'];

// Объединяем документы и запрос в один датасет
$dataset = new Unlabeled(array_merge($documents, $query));

// Преобразуем тексты в TF-IDF векторы
$tfidf = new TfIdfTransformer();
$tfidf->fit($dataset);
$dataset->apply($tfidf);

// Получаем числовые векторы
$vectors = $dataset->samples();

// Последний вектор — это запрос
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

......

#### Что здесь происходит

1. **Преобразование текста в векторы**\
   С помощью `TfIdfTransformer` каждый текст переводится в числовое представление.
2. **Сравнение с запросом**\
   Для каждого документа считается косинусное сходство с вектором запроса.
3. **Сортировка результатов**\
   Документы упорядочиваются по убыванию сходства — наиболее релевантные оказываются первыми.

#### Вывод

Даже если документы имеют разную длину, косинусное сходство позволяет корректно сравнивать их по смыслу. Именно поэтому такой подход лежит в основе большинства систем полнотекстового и семантического поиска.
