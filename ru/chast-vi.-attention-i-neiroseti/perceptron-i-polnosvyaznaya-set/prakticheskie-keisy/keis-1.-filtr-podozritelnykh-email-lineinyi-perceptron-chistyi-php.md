# Кейс 1. Фильтр подозрительных email – линейный перцептрон (чистый PHP)

Задача

Определить, является ли письмо потенциально фишинговым на основе простых числовых признаков.

Признаки

Пусть мы извлекаем:\
• x₁ – количество ссылок в письме\
• x₂ – количество слов-триггеров (“urgent”, “verify”, “password”)\
• x₃ – есть ли внешние домены (0/1)\
• x₄ – длина письма

Это уже числовой вектор.

⸻

Почему это перцептрон?

Если фишинговые письма линейно отделимы по этим признакам, простая гиперплоскость может их разделить.

⸻

Минимальная реализация

```
<?php

class Perceptron
{
    private array $w;
    private float $b = 0.0;
    private float $lr;

    public function __construct(int $nFeatures, float $lr = 0.1)
    {
        $this->w = array_fill(0, $nFeatures, 0.0);
        $this->lr = $lr;
    }

    public function predict(array $x): int
    {
        $z = $this->b;
        foreach ($x as $i => $value) {
            $z += $this->w[$i] * $value;
        }

        return $z > 0 ? 1 : 0;
    }

    public function train(array $X, array $y, int $epochs = 50): void
    {
        for ($e = 0; $e < $epochs; $e++) {
            foreach ($X as $i => $x) {
                $pred = $this->predict($x);
                $error = $y[$i] - $pred;

                foreach ($x as $j => $value) {
                    $this->w[$j] += $this->lr * $error * $value;
                }

                $this->b += $this->lr * $error;
            }
        }
    }
}
```

### Вывод

* Перцептрон – это линейный фильтр
* Он быстро работает
* Он объясним: можно показать вклад каждого признака

<br>

Для B2B-продукта по awareness это отличный baseline-модуль.

\
\
<br>
