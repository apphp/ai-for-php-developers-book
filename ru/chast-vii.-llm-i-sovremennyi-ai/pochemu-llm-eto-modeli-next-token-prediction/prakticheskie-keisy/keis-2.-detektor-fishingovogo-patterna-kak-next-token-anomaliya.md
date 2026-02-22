# Кейс 2. Детектор фишингового паттерна как next-token аномалия

#### Идея

Мы обучаем модель на нормальных письмах компании.

Если в письме возникает токен с низкой условной вероятностью:

P(token \mid context) \approx 0

– это аномалия.

#### Пример

Нормальные письма:

```
Please review the document
Meeting is scheduled tomorrow
Let me know if you have questions
```

Фишинговый текст:

```
Please verify your password immediately
```

Если токен password редко встречается после your, вероятность будет низкой.

#### Логика на PHP

```
function anomalyScore(array $tokens, array $model): float {
    $score = 0.0;

    for ($i = 0; $i < count($tokens) - 1; $i++) {
        $current = $tokens[$i];
        $next = $tokens[$i + 1];

        $prob = $model[$current][$next] ?? 1e-6;

        $score += -log($prob);
    }

    return $score;
}
```

Это упрощённый аналог cross-entropy.

### Что это показывает

LLM оценивает:

P(token \mid context)

Если вероятность низкая – возникает “подозрение”.

Ты можешь встроить это как объяснимую модель в awareness-продукт.
