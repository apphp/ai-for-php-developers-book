# Пример 1. Ранжирование ответов (Preference Learning)

Это самый важный кусок RLHF, и его проще всего объяснить.

Идея: модель учится не абсолютной оценке, а выбору лучшего варианта.

#### Пример на PHP

```
<?php

function scoreAnswer(string $answer): int {
    $score = 0;

    // длиннее — чаще полезнее
    if (strlen($answer) > 80) {
        $score += 1;
    }

    // наличие структуры
    if (str_contains($answer, "1.") || str_contains($answer, "шаг")) {
        $score += 2;
    }

    // наличие объяснения
    if (str_contains($answer, "потому что")) {
        $score += 1;
    }

    return $score;
}

function chooseBetter(string $a, string $b): string {
    return scoreAnswer($a) >= scoreAnswer($b) ? $a : $b;
}

// тест
$q = "Как выучить PHP?";

$a = "Читайте документацию.";
$b = "1. Изучите синтаксис\n2. Пишите код каждый день\n3. Делайте проекты, потому что практика важна";

$best = chooseBetter($a, $b);

echo "Лучший ответ:\n$best\n";
```

***

#### Что здесь важно понять

<br>

Вы фактически делаете то же самое, что и в RLHF:

<br>

– есть несколько ответов

– есть критерий качества

– выбирается лучший

\
Это и есть ядро обучения reward-модели.

