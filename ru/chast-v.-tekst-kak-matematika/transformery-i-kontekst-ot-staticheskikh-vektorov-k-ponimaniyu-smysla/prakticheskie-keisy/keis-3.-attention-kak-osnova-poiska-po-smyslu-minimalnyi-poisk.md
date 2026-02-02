# Кейс 3. Attention как основа поиска по смыслу (минимальный поиск)

Идея

Это мостик к следующим главам: semantic search, clustering, QA.

Сценарий

Есть набор коротких описаний, и пользовательский запрос.

```
$documents = [
    'Как безопасно хранить API ключи',
    'Как поменять замок входной двери',
    'Основы аутентификации и токенов',
    'Как выбрать надёжный дверной замок',
];

$query = 'безопасность ключей доступа';
```

PHP-код

```
$docEmbeddings = $model->embed($documents);
$queryEmbedding = $model->embed([$query])[0];

$scores = [];

foreach ($docEmbeddings as $i => $embedding) {
    $scores[$documents[$i]] = cosineSimilarity($queryEmbedding, $embedding);
}

arsort($scores);
print_r($scores);
```

Результат

Наверху оказываются тексты про API и токены, а не про двери.

Ключевой вывод для этой главы

> Мы сравниваем не слова, а контексты. Attention – фундамент, на котором это вообще возможно.
