# Кейс 2. Фильтр фишинговых писем – MLPClassifier в RubixML

#### Цель кейса

В предыдущем кейсе мы реализовали один нейрон вручную на чистом PHP.

Мы увидели полный цикл:

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

Но один нейрон — это только начало.

В реальных задачах, например в системах информационной безопасности, одного признака недостаточно.

Фишинговое письмо определяется не одним фактором, а комбинацией признаков:

* есть ли ссылка;
* неизвестен ли отправитель;
* используется ли срочный язык;
* запрашивается ли пароль;
* есть ли вложение.

Для таких задач используются многослойные нейронные сети.

В этом кейсе мы создадим простой фильтр фишинговых писем с помощью:

* PHP;
* RubixML;
* многослойного перцептрона (`MLPClassifier`).

Главная цель кейса:

Понять, как теория backpropagation из предыдущего раздела превращается в реальный ML-код.

***

## Сценарий: обнаружение фишинговых писем

Представим, что мы разрабатываем B2B security-платформу.

Каждый день компания получает тысячи писем.

Нужно автоматически определить:

```
0 — безопасное письмо

1 — потенциальный phishing
```

Пример:

Письмо:

```
Your password expires today.
Click here immediately.
```

Признаки:

| Признак                 | Значение |
| ----------------------- | -------- |
| Есть ссылка             | 1        |
| Неизвестный отправитель | 1        |
| Есть срочные слова      | 1        |
| Запрос пароля           | 1        |
| Есть вложение           | 0        |

Вектор:

```
[
  1,
  1,
  1,
  1,
  0
]
```

Модель должна вернуть:

```
phishing = 0.96
```

***

## Общая архитектура решения

Полный pipeline:

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

Разберем каждый этап.

***

## 1. Подготовка Dataset

Любая ML-модель работает не с письмами напрямую.

Нейронная сеть не понимает:

```
"Your password expires today"
```

Она работает только с числами.

Поэтому сначала нужно преобразовать письмо в набор признаков.

***

## 2. Feature extraction

Feature extraction — это процесс извлечения признаков.

Мы превращаем текст письма в числовой вектор:

Было:

```
Dear user,

Your password expires today.
Click here.
```

Стало:

```
[
    1,
    1,
    1,
    1,
    0
]
```

Где:

```
has_link

есть ли ссылка


unknown_sender

неизвестен ли отправитель


urgent_words

есть ли слова:

urgent
today
immediately


password_request

есть ли запрос пароля


attachment

есть ли вложение
```

***

### Простой FeatureExtractor

В реальном продукте здесь могут использоваться:

* NLP-модели;
* embeddings;
* анализ URL;
* анализ домена;
* поведенческие признаки.

Но для понимания backpropagation сделаем простой вариант.

Создадим класс:

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
        return str_contains($text, 'http')
            ? 1
            : 0;
    }


    private function hasUrgentWords(string $text): int
    {
        $words = [
            'urgent',
            'immediately',
            'today'
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
        )
            ? 1
            : 0;
    }
}
```

***

## 3. Создание Dataset для RubixML

RubixML работает с объектами Dataset.

У нас есть два типа данных:

### Samples

Это входные признаки:

```php
[
    1,
    1,
    1,
    1,
    0
]
```

### Labels

Это правильный ответ:

```
phishing
```

или

```
safe
```

***

Создаем обучающие данные:

```php
$samples = [

    [
        1,
        1,
        1,
        1,
        0
    ],


    [
        0,
        0,
        0,
        0,
        0
    ],


    [
        1,
        0,
        1,
        1,
        1
    ],


    [
        0,
        1,
        0,
        0,
        0
    ]

];


$labels = [

    'phishing',

    'safe',

    'phishing',

    'safe'

];
```

***

Теперь создаем Dataset:

```php
use Rubix\ML\Datasets\Labeled;


$dataset = new Labeled(
    $samples,
    $labels
);
```

***

## 4. Архитектура нейросети

Теперь создадим модель.

Мы используем:

```
MLPClassifier
```

MLP означает:

```
Multi Layer Perceptron
```

То есть многослойный перцептрон.

Наша сеть:

```
Input layer

5 признаков

     ↓

Hidden layer

8 нейронов

     ↓

Hidden layer

4 нейрона

     ↓

Output

phishing probability
```

Схематично:

```
                 x1
                 |
                 x2
                 |
                 x3      Dense(8)
                 |
                 x4
                 |
                 x5

                    ↓

              Dense(4)

                    ↓

              Output
```

***

## Создание модели RubixML

Устанавливаем пакет:

```bash
composer require rubix/ml
```

Подключаем:

```php
use Rubix\ML\Classifiers\MLPClassifier;
use Rubix\ML\NeuralNet\Optimizers\Adam;
use Rubix\ML\NeuralNet\CostFunctions\CrossEntropy;
```

***

Создаем сеть:

```php
$estimator = new MLPClassifier(
    [
        8,
        4
    ],
    100,
    new Adam(),
    0.01,
    new CrossEntropy()
);
```

***

Что означают параметры:

#### Скрытые слои

```php
[
    8,
    4
]
```

Это:

```
5 inputs

↓

8 neurons

↓

4 neurons

↓

output
```

***

#### Количество эпох

```php
100
```

Одна эпоха — один полный проход по обучающим данным.

***

#### Optimizer

```php
new Adam()
```

Это алгоритм обновления весов.

Он делает то же самое, что мы делали вручную:

```
weight =
weight - learning_rate * gradient
```

***

#### Loss function

```php
CrossEntropy
```

Она оценивает ошибку классификации.

***

## 5. Обучение модели

Теперь запускаем обучение:

```php
$estimator->train($dataset);
```

На этом месте происходит магия.

Но теперь мы понимаем, что внутри:

```
train()

↓

forward pass

↓

prediction

↓

calculate loss

↓

backpropagation

↓

calculate gradients

↓

update weights

↓

repeat
```

***

## Где происходит backpropagation?

Это самый важный момент главы.

В нашем первом кейсе мы писали:

```php
$neuron->backward();
```

сами.

В RubixML это скрыто внутри:

```
MLPClassifier

        |
        |
        v

Neural Network layers

        |
        |
        v

backward()

        |
        |
        v

Optimizer

        |
        |
        v

new weights
```

Когда вызывается:

```php
$estimator->train($dataset);
```

RubixML автоматически:

1. делает forward pass;
2. считает ошибку;
3. вычисляет градиенты;
4. распространяет ошибку назад;
5. изменяет веса.

То есть backpropagation никуда не исчез.

Он просто находится внутри библиотеки.

***

## 6. Prediction

После обучения модель можно использовать.

Новое письмо:

```php
$email = [

    'body' =>
    'Your password expires today.
     Click http://fake-login.com',

    'unknown_sender' => true,

    'attachment' => false

];
```

Получаем признаки:

```php
$features =
    $extractor->extract($email);
```

Результат:

```php
[
    1,
    1,
    1,
    1,
    0
]
```

Теперь отправляем в модель:

```php
$result =
    $estimator->predict(
        [
            $features
        ]
    );


echo $result[0];
```

***

Получаем:

```
phishing
```

***

## Вероятность риска

Для security-систем обычно нужен не только класс:

```
phishing
```

а вероятность:

```
95% phishing
```

Например:

```php
$probabilities =
    $estimator->proba(
        [
            $features
        ]
    );


print_r(
    $probabilities
);
```

Результат:

```
[
    safe => 0.04,
    phishing => 0.96
]
```

Теперь можно принимать решение:

```php
if ($probabilities['phishing'] > 0.9) {

    echo "Block email";

}
```

***

## Что произошло внутри нейросети

Мы начали с:

```
Email
```

Затем:

```
Email

↓

5 числовых признаков

↓

нейросеть

↓

prediction

↓

ошибка

↓

backpropagation

↓

новые веса
```

Каждый нейрон внутри сети делал то же самое:

```
z = wx+b

activation

error

gradient

update
```

***

## Выводы

В этом кейсе мы построили настоящий ML pipeline:

✅ подготовили Dataset;\
✅ сделали Feature extraction;\
✅ создали многослойную сеть;\
✅ обучили MLPClassifier;\
✅ сделали prediction;\
✅ разобрали, где работает backpropagation.

Главная идея:

RubixML не заменяет понимание backpropagation. Он просто автоматизирует тот процесс, который мы реализовали вручную в предыдущем кейсе.

В первом кейсе у нас был:

```
Neuron
```

Во втором:

```
Neuron
Neuron
Neuron
...
+
Backpropagation
+
Optimizer
```

Но математический принцип остался тем же.

***

## Практическая связь с B2B Security

В реальной платформе Awareness Security этот pipeline будет выглядеть сложнее:

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

Но фундамент остается прежним:

сеть делает прогноз, измеряет ошибку и через backpropagation улучшает свои веса.
