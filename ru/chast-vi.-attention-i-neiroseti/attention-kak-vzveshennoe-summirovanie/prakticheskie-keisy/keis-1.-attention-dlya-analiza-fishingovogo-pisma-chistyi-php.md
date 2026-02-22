# Кейс 1. Attention для анализа фишингового письма (чистый PHP)

#### Цель

Понять, какие слова в письме влияют на финальный “риск-скор”.

#### Сценарий

У нас есть письмо:

> “Your account has been suspended. Click here to verify immediately.”

Мы хотим:

* посчитать важность слов
* получить итоговый embedding письма
* показать пользователю, какие слова повлияли на риск

#### Идея

Каждое слово → embedding

Self-attention → веса

Взвешенная сумма → итоговый вектор письма

#### Мини-реализация (упрощённо)

```
function attentionWeights(array $queries, array $keys) {
    $scores = [];
    foreach ($queries as $i => $q) {
        foreach ($keys as $j => $k) {
            $scores[$i][$j] = dot($q, $k);
        }
    }
    return softmax2D($scores);
}
```

Дальше:

* сортируем веса
* показываем топ-3 слов

#### Практическая ценность

В B2B-продукте можно:

* визуализировать “почему это фишинг”
* повышать explainability
* строить awareness-отчёты

Это сразу превращает attention из теории в бизнес-инструмент.
