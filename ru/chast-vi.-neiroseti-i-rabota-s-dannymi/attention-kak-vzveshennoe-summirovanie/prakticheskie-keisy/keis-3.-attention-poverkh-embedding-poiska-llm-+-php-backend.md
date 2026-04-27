# Кейс 3. Attention поверх embedding-поиска (LLM + PHP backend)

#### Цель

Сделать умный RAG-поиск.

#### Сценарий

У тебя есть:

* 10 документов
* embedding запроса

Обычно делается cosine similarity → top-k.

Можно улучшить:

* взять top-k
* прогнать attention между query и кандидатами
* получить адаптивные веса

Формально

score\_i = \text{softmax}(Q \cdot K\_i)

output = \sum score\_i \cdot V\_i

#### PHP-скелет

```
function rerankWithAttention($queryVec, $docVecs) {
    $scores = [];

    foreach ($docVecs as $doc) {
        $scores[] = dot($queryVec, $doc);
    }

    $weights = softmax($scores);

    return $weights;
}
```

#### Практическая ценность

* Улучшенный search
* Более точная генерация ответов
* Можно встроить в EventumX для timeline-поиска

