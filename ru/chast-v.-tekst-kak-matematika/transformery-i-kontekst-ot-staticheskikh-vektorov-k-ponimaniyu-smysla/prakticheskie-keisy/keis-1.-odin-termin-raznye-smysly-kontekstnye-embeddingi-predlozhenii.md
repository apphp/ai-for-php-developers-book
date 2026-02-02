# Кейс 1. Один термин – разные смыслы (контекстные эмбеддинги предложений)

Идея

Мы не сравниваем слова.

Мы сравниваем эмбеддинги предложений, где одно и то же слово встречается в разных контекстах.

Это самый точный кейс под эту главу. Он буквально материализует фразу\
«почему слово "ключ" имеет разные векторы».

Сценарий

Берём несколько предложений:

– «Он потерял ключ от квартиры»

– «API использует секретный ключ»

– «Ключевая идея доклада была неожиданной»

Строим эмбеддинги предложений и считаем расстояния.

Реальный PHP-код (через готовую модель)

Если использовать transformers-php или обёртку над ONNX:

```
use function Codewithkyrian\Transformers\Pipelines\pipeline;

// pipeline для извлечения эмбеддингов
$embedder = pipeline(
    task: 'feature-extraction',
    model: 'sentence-transformers/all-MiniLM-L6-v2'
);

$sentences = [
    'Он потерял ключ от квартиры',
    'API использует секретный ключ',
    'Ключевая идея доклада была неожиданной',
];

// получаем эмбеддинги
$embeddings = $embedder($sentences);

// обычно возвращается массив вида [sentence][token][dim]
// усредняем по токенам (mean pooling)
function meanPooling(array $tokenEmbeddings): array {
    $sum = array_fill(0, count($tokenEmbeddings[0]), 0.0);

    foreach ($tokenEmbeddings as $token) {
        foreach ($token as $i => $value) {
            $sum[$i] += $value;
        }
    }

    return array_map(
        fn($v) => $v / count($tokenEmbeddings),
        $sum
    );
}

$vectors = array_map('meanPooling', $embeddings);

// cosine similarity
function cosineSimilarity(array $a, array $b): float {
    $dot = $normA = $normB = 0.0;

    foreach ($a as $i => $v) {
        $dot += $v * $b[$i];
        $normA += $v ** 2;
        $normB += $b[$i] ** 2;
    }

    return $dot / (sqrt($normA) * sqrt($normB));
}

echo cosineSimilarity($vectors[0], $vectors[1]) . PHP_EOL;
echo cosineSimilarity($vectors[0], $vectors[2]) . PHP_EOL;

```

Что мы видим

– предложения про API и квартиру далеки друг от друга

– «ключевая идея» тоже уходит в другую сторону

Ключевой вывод

> Контекст задаёт геометрию. Слово «ключ» не имеет собственного смысла вне предложения.

<br>
