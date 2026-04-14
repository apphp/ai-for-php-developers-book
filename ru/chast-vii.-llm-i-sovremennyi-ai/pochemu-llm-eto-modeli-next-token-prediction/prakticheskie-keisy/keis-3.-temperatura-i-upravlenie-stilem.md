# Кейс 3. Температура и управление стилем

#### Задача

Показать сотрудникам, как temperature влияет на генерацию.

#### Добавляем temperature в сэмплирование

```
function sampleWithTemperature(array $distribution, float $temperature): string {
    $adjusted = [];
    $sum = 0;

    foreach ($distribution as $token => $prob) {
        $value = pow($prob, 1 / $temperature);
        $adjusted[$token] = $value;
        $sum += $value;
    }

    // нормализация
    foreach ($adjusted as $token => $value) {
        $adjusted[$token] = $value / $sum;
    }

    return sampleNext($adjusted);
}
```

#### Демонстрация

* T = 0.3 → формальный корпоративный стиль
* T = 1.0 → нейтральный
* T = 1.8 → хаотичный

#### Что понимает читатель

LLM не “становится креативной”.

Она просто:

* меняет форму вероятностного распределения.











