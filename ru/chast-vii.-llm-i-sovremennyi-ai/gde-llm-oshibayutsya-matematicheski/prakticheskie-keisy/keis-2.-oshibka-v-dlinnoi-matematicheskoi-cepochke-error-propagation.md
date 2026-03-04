# Кейс 2. Ошибка в длинной математической цепочке (error propagation)

#### Сценарий

Ты создаёшь AI-тьютора по математике.

Модель решает задачу пошагово (chain of thought).

Если ошибка на шаге 2, всё дальнейшее решение логично, но неверно.

Математически:

P(correct\ chain) = \prod\_{t=1}^{n} P(correct\_t)

Если каждый шаг верен с вероятностью 0.95, а шагов 15:

0.95^{15} \approx 0.46

Менее 50%.

#### Решение: автоматическая перепроверка шагов

LLM генерирует шаг → PHP вычисляет формулу → сравниваем.

#### Пример проверки выражений

```
<?php

function evaluateExpression(string $expr): float {
    // крайне упрощённый пример!
    return eval("return $expr;");
}

$llmStep = "5 * (3 + 2) = 30";

preg_match('/(.+)=\s*(\d+)/', $llmStep, $matches);

$expression = trim($matches[1]);
$claimed = (float)$matches[2];

$real = evaluateExpression($expression);

if ($real !== $claimed) {
    echo "Step is incorrect. Claimed: $claimed, Real: $real";
}
```

В production, конечно, eval использовать нельзя – лучше подключить math-parser библиотеку.

#### Инженерный вывод

LLM не должна быть вычислительным движком.

Она должна быть объясняющим интерфейсом поверх точного движка.

