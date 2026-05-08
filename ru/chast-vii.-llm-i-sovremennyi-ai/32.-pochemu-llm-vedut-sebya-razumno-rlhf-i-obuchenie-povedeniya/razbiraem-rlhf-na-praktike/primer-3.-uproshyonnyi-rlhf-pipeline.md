# Пример 3. Упрощённый RLHF pipeline

Теперь собираем всё вместе.

Это уже полноценная симуляция pipeline:

– генерация вариантов

– оценка

– выбор лучшего

***

#### Пример

```
<?php

function generateAnswers(string $question): array {
    return [
        "Попробуйте изучить PHP.",
        
        "Начните с основ PHP и читайте документацию.",
        
        "1. Изучите синтаксис PHP\n2. Практикуйтесь ежедневно\n3. Делайте проекты, например CRUD-приложение"
    ];
}

function reward(string $answer): float {
    $score = 0;

    $score += strlen($answer) / 50;

    if (preg_match('/\d+\./', $answer)) {
        $score += 2;
    }

    if (str_contains($answer, "например")) {
        $score += 1;
    }

    return $score;
}

function optimizeAnswer(string $question): string {
    $candidates = generateAnswers($question);

    $best = null;
    $bestScore = -INF;

    foreach ($candidates as $answer) {
        $score = reward($answer);

        if ($score > $bestScore) {
            $bestScore = $score;
            $best = $answer;
        }
    }

    return $best;
}

// запуск
$q = "Как выучить PHP?";

echo optimizeAnswer($q);
```

***

### Что здесь происходит (это важно проговорить в книге)

Вы фактически реализовали:

– policy → generateAnswers()

– reward model → reward()

– optimization → optimizeAnswer()

Это и есть RLHF в миниатюре.
