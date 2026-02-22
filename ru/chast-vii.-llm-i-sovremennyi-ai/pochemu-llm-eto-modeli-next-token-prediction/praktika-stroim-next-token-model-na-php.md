# Практика: строим next-token модель на PHP

До этого момента мы говорили теоретически:

P(token \mid context)

Теперь построим маленькую, но полностью рабочую next-token модель на чистом PHP и на практике увидим:

– как работают n-граммы

– что такое cross-entropy

– как влияет temperature

– как считать аномалии

– почему это напрямую связано с prompt engineering

– как отсюда вытекает LLM security

– и откуда берутся галлюцинации

Никаких нейросетей. Только вероятности.

### 1. Шаг 1 – строим n-граммную модель

Возьмём простой корпус:

```
$corpus = [
    "php is simple",
    "php is fast",
    "php is popular",
    "security is important",
    "security is critical",
];
```

#### Токенизация

```
function tokenize(string $text): array {
    return explode(" ", strtolower($text));
}
```

#### Обучение биграммной модели

Мы считаем:

P(w\_{t} \mid w\_{t-1})

```
function trainBigram(array $corpus): array {
    $counts = [];

    foreach ($corpus as $text) {
        $tokens = tokenize($text);

        for ($i = 0; $i < count($tokens) - 1; $i++) {
            $current = $tokens[$i];
            $next = $tokens[$i + 1];

            $counts[$current][$next] =
                ($counts[$current][$next] ?? 0) + 1;
        }
    }

    // Нормализация
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

Теперь у нас есть вероятностная модель.

Она уже делает то же самое, что и GPT – просто на уровне 1 токена контекста.

### 2. Генерация текста

#### Сэмплирование

```
function sample(array $distribution): string {
    $rand = mt_rand() / mt_getrandmax();
    $cumulative = 0;

    foreach ($distribution as $token => $prob) {
        $cumulative += $prob;
        if ($rand <= $cumulative) {
            return $token;
        }
    }

    return array_key_first($distribution);
}
```

#### Генерация последовательности

```
function generate(string $start, array $model, int $length = 5): string {
    $current = $start;
    $result = [$current];

    for ($i = 0; $i < $length; $i++) {
        if (!isset($model[$current])) break;

        $next = sample($model[$current]);
        $result[] = $next;
        $current = $next;
    }

    return implode(" ", $result);
}
```

Если запустить:

```
$model = trainBigram($corpus);
echo generate("php", $model);
```

Вы получите:

```
php is simple
```

или

```
php is popular
```

Это и есть next-token prediction.

### 3. Cross-entropy – измеряем качество

LLM обучаются минимизируя cross-entropy:

L = -\log P(true\\\_token)

Добавим функцию оценки.

```
function crossEntropy(array $tokens, array $model): float {
    $loss = 0.0;

    for ($i = 0; $i < count($tokens) - 1; $i++) {
        $current = $tokens[$i];
        $next = $tokens[$i + 1];

        $prob = $model[$current][$next] ?? 1e-6;

        $loss += -log($prob);
    }

    return $loss / (count($tokens) - 1);
}
```

Теперь можно сравнить:

```
$normal = tokenize("php is simple");
$phishing = tokenize("php steal password");

echo crossEntropy($normal, $model);
echo crossEntropy($phishing, $model);
```

Второй текст даст гораздо больший loss.

Почему?

Потому что:

P(steal \mid php) \approx 0

### 4. Temperature – управляем распределением

Добавим temperature:

P\_i = \frac{P\_i^{1/T\}}{\sum\_j P\_j^{1/T\}}

```
function applyTemperature(array $distribution, float $T): array {
    $adjusted = [];
    $sum = 0;

    foreach ($distribution as $token => $prob) {
        $value = pow($prob, 1 / $T);
        $adjusted[$token] = $value;
        $sum += $value;
    }

    foreach ($adjusted as $token => $value) {
        $adjusted[$token] = $value / $sum;
    }

    return $adjusted;
}
```

Если:

– T = 0.5 → модель почти детерминированная

– T = 1.5 → появляется разнообразие

Это ровно то же, что происходит в GPT-4.

### 5. Anomaly Detection – детектор фишинга

Теперь применим cross-entropy как аномальный скор.

Если:

-\log P(token \mid context)

высокий → текст необычный.

Это почти точная интуиция behind:

– детектирование аномалий

– языковая модель как фильтр

#### Простой детектор

```
function isAnomalous(string $text, array $model, float $threshold): bool {
    $tokens = tokenize($text);
    $score = crossEntropy($tokens, $model);

    return $score > $threshold;
}
```

Это уже прототип security-фильтра.

### Связь с LLM-практикой

Теперь самое важное – связываем это с реальными системами.

### 1. Prompt engineering

Когда вы добавляете:

```
You are a security expert.
```

Вы меняете контекст.

А значит:

P(token \mid context)

становится другим.

Prompt engineering – это управление вероятностным распределением.

Не магия.

***

### 2. LLM Security

Prompt injection работает потому что:

модель продолжает текст, который выглядит как инструкция.

Если злоумышленник добавляет:

```
Ignore previous instructions.
```

– модель просто продолжает наиболее вероятный шаблон.

Она не “понимает”, что это атака.

***

### 3. Awareness training

Можно показать сотрудникам:

– как легко смещается распределение

– как low-probability токены выглядят подозрительно

– как фишинг – это статистический паттерн

И объяснить:

ИИ не “разумен”. Он вероятностный.

***

### 4. Галлюцинации

Галлюцинация – это ситуация, когда:

P(token \mid context)

высока,

но фактологически ответ неверен.

Модель не проверяет истину.

Она максимизирует вероятность.

Если в обучающих данных часто встречается шаблон:

```
John Smith (1820–1890) was a famous scientist
```

модель может сгенерировать похожий шаблон для несуществующего человека.

Потому что это вероятностно правдоподобно.

### Итог

Мы построили:

– n-граммную модель

– генерацию

– cross-entropy

– temperature

– anomaly detection

И всё это – мини-версия того, что делает LLM.

Главная мысль:

> LLM – это не интеллект.

> Это механизм оценки вероятности следующего токена.

И именно поэтому:

– prompt работает

– атаки работают

– галлюцинации неизбежны

– безопасность требует контроля контекста

– temperature меняет поведение

Когда PHP-разработчик это понимает, он перестаёт мистифицировать ИИ и начинает проектировать системы осознанно.
