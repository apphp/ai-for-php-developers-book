# Кейс 6. Архитектура “LLM + детерминированный слой”

Это самый важный инженерный паттерн.

#### Схема

1. LLM генерирует черновик
2. PHP валидирует структуру
3. Специализированный модуль считает числа
4. Верификатор проверяет источники
5. Ответ собирается заново

```
<?php

class AIOrchestrator {

    public function handle(string $question): string
    {
        $draft = $this->askLLM($question);

        if ($this->containsMath($draft)) {
            $draft = $this->verifyMath($draft);
        }

        if ($this->containsFacts($draft)) {
            $draft = $this->verifyFacts($draft);
        }

        return $draft;
    }

    private function askLLM($q) { return "..."; }
    private function containsMath($t) { return true; }
    private function verifyMath($t) { return "Verified version"; }
    private function containsFacts($t) { return false; }
    private function verifyFacts($t) { return $t; }
}
```

#### Финальный инженерный вывод

LLM ошибаются математически не случайно.

Это прямое следствие:

\text{Оптимизация правдоподобия} \neq \text{Оптимизация истины}

Поэтому зрелая AI-система на PHP должна:

– отделять генерацию от вычислений

– учитывать базовые вероятности

– бороться со смещением данных

– контролировать temperature

– внедрять верификацию

– использовать классические ML-модели там, где нужна строгая статистика



