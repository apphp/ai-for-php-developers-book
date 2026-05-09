# Кейс 1. Умное автодополнение письма (B2B awareness)

#### Задача

Внутренний корпоративный редактор письма предсказывает вероятное продолжение фразы сотрудника.

Цель:

– показать, как модель “подталкивает” к определённому стилю

– продемонстрировать, что это просто next-token prediction

#### Логика

Мы обучаем простую n-граммную модель на реальных корпоративных письмах.

```
Dear team
Please find attached
Let me know
Best regards
```

Мы считаем:

P(next\\\_token \mid previous\\\_token)

#### Минимальная реализация на чистом PHP

#### Обучение биграммной модели

```
function trainBigramModel(array $texts): array {
    $counts = [];

    foreach ($texts as $text) {
        $tokens = explode(" ", strtolower($text));

        for ($i = 0; $i < count($tokens) - 1; $i++) {
            $current = $tokens[$i];
            $next = $tokens[$i + 1];

            $counts[$current][$next] = ($counts[$current][$next] ?? 0) + 1;
        }
    }

    // Превращаем в вероятности
    $model = [];

    foreach ($counts as $token => $nextTokens) {
        $total = array_sum($nextTokens);

        foreach ($nextTokens as $next => $count) {
            $model[$token][$next] = $count / $total;
        }
    }

    return $model;
}
```

#### Предсказание следующего токена

```
function predictNextToken(string $token, array $model): ?string {
    if (!isset($model[$token])) return null;

    arsort($model[$token]); // берём самый вероятный

    return array_key_first($model[$token]);
}
```

#### Что демонстрирует кейс

* автодополнение – это просто выбор самого вероятного токена
* стиль текста – это статистика
* LLM делает то же самое, но на миллиардах токенов



