---
description: Создаем многослойную нейронную сеть для классификации подозрительных писем.
---

# Кейс 2. Фильтр фишинговых писем – MLPClassifier в RubixML

В предыдущем кейсе мы реализовали один нейрон вручную на чистом PHP.

Мы увидели полный цикл обучения:

```
input
  ↓
forward pass
  ↓
prediction
  ↓
loss
  ↓
backpropagation
  ↓
update weights
```

Один нейрон хорошо показывает основную идею обучения.

Но в реальных задачах одного признака или одного нейрона часто недостаточно. Например, фишинговое письмо определяется не одним фактором, а комбинацией признаков:

* есть ли ссылка;
* известен ли отправитель;
* используются ли срочные слова;
* запрашивается ли пароль;
* есть ли вложение.

Для таких задач можно использовать многослойную нейронную сеть.

В этом кейсе мы создадим простой фильтр фишинговых писем с помощью:

* PHP;
* RubixML;
* многослойного перцептрона (`MLPClassifier`).

Главная цель – увидеть, как теория backpropagation из предыдущего кейса превращается в реальный ML-код.

***

### Сценарий: обнаружение фишинговых писем

Представим, что мы разрабатываем B2B security-платформу.

Компания получает тысячи писем каждый день. Нам нужно автоматически определить, является ли письмо безопасным или потенциально фишинговым.

Будем использовать два класса:

```
0 – safe
1 – phishing
```

Например:

```
Your password expires today.

Click here immediately.
```

можно представить следующими признаками:

| Признак                 | Значение |
| ----------------------- | -------: |
| Есть ссылка             |        1 |
| Неизвестный отправитель |        1 |
| Есть срочные слова      |        1 |
| Запрашивается пароль    |        1 |
| Есть вложение           |        0 |

Получаем вектор:

```
[1, 1, 1, 1, 0]
```

Общий pipeline будет выглядеть так:

```
Email
  ↓
Feature extraction
  ↓
Dataset
  ↓
MLPClassifier
  ↓
Training
  ↓
Prediction
  ↓
Risk decision
```

Разберём его по шагам.

***

### 1. Feature extraction

Нейронная сеть не понимает письмо в виде текста:

```
Your password expires today.
Click here immediately.
```

Ей нужны числовые данные.

Поэтому сначала преобразуем письмо в набор признаков:

```
has_link
unknown_sender
urgent_words
password_request
attachment
```

Например:

```
[1, 1, 1, 1, 0]
```

Это называется **Feature Extraction** – извлечение признаков из исходных данных.

Для учебного примера сделаем простой extractor:

```php
class EmailFeatureExtractor
{
    public function extract(array $email): array
    {
        return [
            $this->hasLink($email['body']),
            $email['unknown_sender'] ? 1 : 0,
            $this->hasUrgentWords($email['body']),
            $this->hasPasswordRequest($email['body']),
            $email['attachment'] ? 1 : 0,
        ];
    }

    private function hasLink(string $text): int
    {
        return str_contains($text, 'http') ? 1 : 0;
    }

    private function hasUrgentWords(string $text): int
    {
        $words = [
            'urgent',
            'immediately',
            'today',
        ];

        foreach ($words as $word) {
            if (stripos($text, $word) !== false) {
                return 1;
            }
        }

        return 0;
    }

    private function hasPasswordRequest(string $text): int
    {
        return str_contains(
            strtolower($text),
            'password'
        ) ? 1 : 0;
    }
}
```

Теперь можем обработать письмо:

```php
$email = [
    'body' =>
        'Your password expires today. '
        . 'Click http://fake-login.example',
    'unknown_sender' => true,
    'attachment' => false,
];

$extractor = new EmailFeatureExtractor();

$features = $extractor->extract($email);
```

Получим:

```
[1, 1, 1, 1, 0]
```

***

### 2. Создание Dataset

RubixML работает с объектами Dataset.

У нас есть два типа данных:

* **samples** – входные признаки;
* **labels** – правильные ответы.

Например:

```
Sample:

[1, 1, 1, 1, 0]

Label:

phishing
```

Создадим небольшой учебный dataset:

```php
$samples = [
    [1, 1, 1, 1, 0],
    [0, 0, 0, 0, 0],
    [1, 0, 1, 1, 1],
    [0, 1, 0, 0, 0],
];

$labels = [
    'phishing',
    'safe',
    'phishing',
    'safe',
];
```

Теперь создадим `Labeled` dataset:

```php
use Rubix\ML\Datasets\Labeled;

$dataset = new Labeled(
    $samples,
    $labels
);
```

Наши данные теперь готовы для обучения модели.

***

### 3. Архитектура нейронной сети

Теперь создадим многослойный перцептрон.

У нас пять входных признаков:

```
x1 = has_link
x2 = unknown_sender
x3 = urgent_words
x4 = password_request
x5 = attachment
```

Архитектура сети:

```
5 inputs
   ↓
8 neurons
   ↓
4 neurons
   ↓
output
```

Или кратко:

```
5 → 8 → 4 → output
```

Схематично:

```
x1 ─┐
x2 ─┤
x3 ─┼──→ Dense(8)
x4 ─┤       ↓
x5 ─┘     Dense(4)
           ↓
         Output
```

Мы используем `MLPClassifier`.

Устанавливаем RubixML:

```bash
composer require rubix/ml
```

И подключаем необходимые классы:

```php
use Rubix\ML\Classifiers\MLPClassifier;
use Rubix\ML\NeuralNet\Optimizers\Adam;
use Rubix\ML\NeuralNet\CostFunctions\CrossEntropy;
```

Создаём модель:

```php
$estimator = new MLPClassifier(
    [8, 4],
    100,
    new Adam(),
    0.01,
    new CrossEntropy()
);
```

Здесь:

```
[8, 4]
```

означает два скрытых слоя:

```
5 → 8 → 4 → output
```

`100` – количество эпох обучения.

`Adam` – optimizer, который обновляет веса.

`0.01` – learning rate.

`CrossEntropy` – функция потерь для классификации.

> Конкретный API MLP зависит от версии RubixML. Если вы используете другую версию библиотеки, сверяйте сигнатуру конструктора с её документацией.

***

### 4. Обучение модели

Теперь запускаем обучение:

```php
$estimator->train($dataset);
```

На первый взгляд здесь происходит всего одна операция.

Но внутри выполняется весь знакомый нам цикл:

```
forward pass
    ↓
prediction
    ↓
loss
    ↓
backpropagation
    ↓
gradients
    ↓
optimizer
    ↓
update weights
```

Этот цикл повторяется во время обучения.

#### Где происходит backpropagation?

В предыдущем кейсе мы сами вызывали:

```php
$neuron->backward();
```

Теперь этот код скрыт внутри ML-библиотеки.

Когда мы пишем:

```php
$estimator->train($dataset);
```

RubixML выполняет необходимые операции внутри нейронной сети:

```
forward pass
      ↓
prediction
      ↓
loss
      ↓
backpropagation
      ↓
gradients
      ↓
optimizer
      ↓
new weights
```

Главная идея:

> **RubixML не заменяет backpropagation – он автоматизирует его.**

Математика остаётся той же, которую мы разбирали в предыдущем кейсе.

***

### 5. Prediction

После обучения модель можно использовать для новых писем.

Создадим письмо:

```php
$email = [
    'body' =>
        'Your password expires today. '
        . 'Click http://fake-login.example',
    'unknown_sender' => true,
    'attachment' => false,
];
```

Извлекаем признаки:

```php
$features = $extractor->extract($email);
```

Получаем:

```
[1, 1, 1, 1, 0]
```

Передаём их модели:

```php
$result = $estimator->predict([
    $features,
]);

echo $result[0];
```

Модель может вернуть:

```
phishing
```

***

### 6. Вероятность класса

В security-системе часто недостаточно знать только класс.

Нас может интересовать вероятность:

```
safe     = 0.04
phishing = 0.96
```

Для этого можно использовать `proba()`:

```php
$probabilities = $estimator->proba([
    $features,
]);
```

Полученный score можно использовать в application logic.

Например:

```php
if ($probabilities['phishing'] >= 0.90) {
    echo 'Block email';
}
```

Важно разделять две задачи:

```
ML model
    ↓
risk score
    ↓
application rules
    ↓
security decision
```

Модель делает prediction.

Приложение принимает решение на основе этого prediction.

***

### 7. Что происходит внутри нейросети?

Посмотрим на полный путь одного письма.

Начинаем с:

```
Email
```

После Feature Extraction:

```
[1, 1, 1, 1, 0]
```

Затем происходит forward pass:

```
[1, 1, 1, 1, 0]
        ↓
     Dense(8)
        ↓
     Dense(4)
        ↓
      Output
```

Модель получает prediction:

```
phishing = 0.96
```

Во время обучения мы также вычисляем loss:

```
prediction
     ↓
    loss
```

Затем backpropagation распространяет ошибку назад:

```
output
  ↓
layer 2
  ↓
layer 1
```

Вычисляются градиенты:

```
∂Loss / ∂W
```

и optimizer обновляет веса:

```
old weights
     ↓
  gradients
     ↓
    Adam
     ↓
new weights
```

Именно так сеть постепенно учится делать более точные predictions.

***

### 8. От одного нейрона к нейронной сети

Теперь можно сравнить два кейса.

В предыдущем кейсе:

```
inputs
  ↓
Neuron
  ↓
prediction
  ↓
loss
  ↓
backpropagation
```

В этом кейсе:

```
inputs
  ↓
Dense(8)
  ↓
Dense(4)
  ↓
Output
  ↓
prediction
  ↓
loss
  ↓
backpropagation
```

Архитектура стала сложнее, но основной принцип не изменился.

На уровне отдельных нейронов всё ещё происходят:

```
z = wx + b
```

затем activation:

```
activation(z)
```

затем prediction и loss.

Во время обучения:

```
loss
  ↓
gradient
  ↓
weight update
```

Это тот же самый принцип, который мы реализовали вручную.

***

### 9. Почему здесь нужны несколько слоёв?

Рассмотрим отдельные признаки.

Наличие ссылки:

```
has_link = 1
```

само по себе не означает phishing.

Неизвестный отправитель:

```
unknown_sender = 1
```

тоже не означает phishing.

Вложение:

```
attachment = 1
```

также не является доказательством.

Но комбинация признаков может быть более информативной:

```
unknown_sender   = 1
has_link         = 1
urgent_words     = 1
password_request = 1
```

Многослойная сеть может учиться находить такие комбинации.

Упрощённо:

```
raw features
     ↓
feature combinations
     ↓
learned representation
     ↓
classification
```

***

### 10. Feature Engineering

В нашем примере мы сами выбрали признаки:

```
has_link
unknown_sender
urgent_words
password_request
attachment
```

Это называется **Feature Engineering**.

Мы заранее решили, какая информация может быть полезна модели.

В реальной системе признаков было бы намного больше:

* URL и домены;
* reputation отправителя;
* возраст домена;
* HTML-структура;
* количество ссылок;
* вложения;
* текстовые признаки;
* NLP features;
* embeddings;
* история взаимодействия с отправителем.

Например, вместо простого:

```
has_link = 1
```

можно анализировать сам URL и его домен.

Вместо:

```
urgent_words = 1
```

можно использовать NLP-модель или embeddings.

Но общий pipeline остаётся тем же:

```
Email
  ↓
Feature extraction
  ↓
Numerical features
  ↓
ML model
  ↓
Risk score
```

***

### 11. Training и Prediction

Важно различать обучение и использование модели.

#### Training

Во время обучения выполняется:

```
input
  ↓
forward pass
  ↓
prediction
  ↓
loss
  ↓
backpropagation
  ↓
weight update
```

Здесь веса изменяются.

#### Prediction

После обучения:

```
input
  ↓
forward pass
  ↓
prediction
```

Backpropagation больше не нужен.

Модель просто использует уже обученные веса.

Поэтому:

```
Training  = forward + backward

Inference = forward only
```

***

### 12. Ограничения учебного примера

Наш dataset очень маленький:

```php
$samples = [
    [1, 1, 1, 1, 0],
    [0, 0, 0, 0, 0],
    [1, 0, 1, 1, 1],
    [0, 1, 0, 0, 0],
];
```

Он нужен только для демонстрации.

Для production-системы этого недостаточно.

Нужны:

* большой dataset;
* train/test split;
* validation data;
* разнообразные phishing emails;
* безопасные письма;
* обработка class imbalance;
* monitoring;
* регулярное переобучение;
* анализ ошибок модели.

Особенно важно контролировать два типа ошибок:

**False Positive**

Безопасное письмо классифицировано как phishing.

**False Negative**

Фишинговое письмо классифицировано как safe.

Для security-системы обе ошибки важны, но их последствия могут быть разными.

***

### 13. Метрики

Для оценки такой модели одной accuracy может быть недостаточно.

Например, если почти все письма безопасны, модель может получить высокую accuracy, просто часто выбирая `safe`.

Поэтому полезно смотреть на:

* precision;
* recall;
* F1-score;
* confusion matrix.

В следующем разделе мы подробнее разберём эти метрики и увидим, почему для security-задач важно понимать не только количество правильных ответов, но и тип ошибок.

***

## Выводы

В этом кейсе мы построили полный ML pipeline для классификации фишинговых писем.

Мы:

* преобразовали письмо в числовые признаки;
* создали Dataset;
* построили многослойную сеть;
* обучили `MLPClassifier`;
* сделали prediction;
* получили probability;
* связали prediction с application logic;
* увидели, где внутри ML-библиотеки происходит backpropagation.

Главная идея:

> **ML-библиотека скрывает детали реализации, но не меняет фундаментальную математику.**

В первом кейсе мы работали с одним нейроном:

```
Neuron
  ↓
loss
  ↓
backpropagation
  ↓
update weights
```

Теперь у нас есть:

```
Neuron
Neuron
Neuron
...
  ↓
Neural Network
  ↓
Backpropagation
  ↓
Optimizer
```

Но принцип остался тем же.

***

### Практическая связь с B2B Security

В реальной security-платформе pipeline может выглядеть так:

```
Email
  ↓
NLP / embeddings
  ↓
Feature extraction
  ↓
Neural Network
  ↓
Risk score
  ↓
Security decision
```

Наш учебный пример значительно проще, чем production-система.

Но фундамент тот же:

```
data
  ↓
features
  ↓
model
  ↓
prediction
  ↓
loss
  ↓
backpropagation
  ↓
better weights
```

Именно это мы хотели увидеть в этом кейсе: **как знания о backpropagation превращаются из математической теории в работающий ML-код на PHP.**

