# Кейс 1. Галлюцинация числовых фактов и обязательная верификация

#### Сценарий

Ты строишь AI-ассистента для финансового SaaS. Пользователь спрашивает:

“Сколько было выручки у компании X в 2024 году?”

LLM уверенно генерирует число. Но оно может быть синтезировано на основе паттернов, а не реальных данных.

Вспомним формулу:

\hat{x} = \arg\max\_x P(x \mid context)

Модель выбирает наиболее вероятное число, а не проверенное.

#### Архитектурное решение: генерация + строгая проверка

Мы разделяем:

1. Генерацию ответа
2. Верификацию фактов через API / БД

#### Пример на pure PHP

```
<?php

function askLLM(string $question): string {
    // здесь вызов OpenAI / другого LLM API
    return "Revenue of Company X in 2024 was 2.4 billion USD.";
}

function extractNumber(string $text): ?float {
    if (preg_match('/([\d\.]+)\s*billion/i', $text, $matches)) {
        return (float)$matches[1] * 1_000_000_000;
    }
    return null;
}

function verifyRevenue(float $value): bool {
    // имитация запроса к реальному источнику
    $realValue = 2_100_000_000; 
    return abs($realValue - $value) < 50_000_000;
}

$answer = askLLM("Revenue of Company X in 2024?");
$value = extractNumber($answer);

if ($value !== null && !verifyRevenue($value)) {
    echo "LLM hallucination detected.\n";
} else {
    echo $answer;
}
```

#### Вывод

LLM без внешней проверки – это probabilistic guesser.

В финансовых, юридических, медицинских сценариях верификация обязательна.
