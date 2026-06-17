---
description: Sequence labeling, BIO-разметка.
---

# 5.6 Named Entity Recognition (NER) – извлечение сущностей из текста

До этого момента мы научились превращать слова в числа, числа – в векторы, а векторы – в осмысленные пространства. Теперь сделаем следующий шаг.

Представим текст:

> "Apple bought a startup in London for $3 million."

Человек мгновенно видит структуру:

* Apple – организация
* London – география
* $3 million – денежная сумма

Модель же видит последовательность токенов. Задача NER – научить модель выделять сущности внутри текста.

Это уже не просто классификация текста. \
Это классификация каждого токена в последовательности.

### Что такое NER математически

Пусть текст – это последовательность токенов:

$$
x = (x_1, x_2, ..., x_n)
$$

Наша цель – предсказать последовательность меток:

$$
y = (y_1, y_2, ..., y_n)
$$

Каждый $$y_i$$ – класс токена, например:

* PERSON
* ORG
* LOC
* MONEY
* O (не сущность)
* и т.д.

Модель оценивает:

$$
P(y_1, y_2, ..., y_n \mid x_1, ..., x_n)
$$

В простейшем варианте – независимо по каждому токену:

$$
P(y_i \mid x)
$$

Но уже видно проблему: метка токена зависит от контекста.

Слово "Apple" может быть:

* фруктом
* компанией

Контекст решает всё.

### Почему Bag of Words здесь не работает

Bag of Words уничтожает порядок слов.

NER же – это задача, где порядок критичен.

Фраза:

> "Bank of America"

Если смотреть на слова по отдельности,

"Bank" – это организация,

"America" – география.

Но вместе – это одна сущность ORG.

Поэтому NER требует моделей, которые учитывают последовательность: RNN, LSTM, Transformers. Сегодня – чаще всего Transformers.

### BIO-разметка

Чтобы модель понимала границы сущностей, используется схема BIO (это по сути стандарт, хотя и не все модели её поддерживают):

* B-XXX (Beginning) – начало сущности
* I-XXX (Inside) – продолжение
* O (Outside) – вне сущности

Допустим, у нас есть текст:&#x20;

> "Apple bought a startup in London for $3 million from John Smith"

BIO разметка по нему покажет следующий результат:

| Токен   | Метка   |
| ------- | ------- |
| Apple   | B-ORG   |
| bought  | O       |
| a       | O       |
| startup | O       |
| in      | O       |
| London  | B-LOC   |
| for     | O       |
| $3      | B-MONEY |
| million | I-MONEY |
| from    | O       |
| John    | B-PER   |
| Smith   | I-PER   |

Это уже не просто классификация. Это sequence labeling.

<div align="left"><figure><img src="../../.gitbook/assets/5.6-1_bio-tags.png" alt=""><figcaption><p>Рис. 5.6-1. BIO тэги</p></figcaption></figure></div>

### Sequence labeling как вероятностная задача

Наивный подход:

$$
y_i = \arg\max_k P(y_i = k \mid x)
$$

Но тогда возможны нелепые последовательности:

* I-ORG без B-ORG
* I-MONEY после O

Поэтому продвинутые модели (например, CRF) учитывают:

$$
P(y_i \mid y_{i-1}, x)
$$

Это добавляет структуру в предсказание.

Трансформеры часто комбинируют:

* contextual embeddings
* линейный классификатор
* иногда CRF-слой сверху

### Архитектура современной NER-модели

Схема:

1. Текст → токенизация
2. Токены → эмбеддинги
3. Трансформер → контекстуальные векторы
4. Линейный слой → logits
5. Softmax → вероятности классов

$$
\text{logits}_i = W h_i + b
$$

и

$$
P(y_i) = \text{softmax}(\text{logits}_i)
$$

Где $$h_i$$ – контекстный вектор токена.

<div align="left"><figure><img src="../../.gitbook/assets/5.6-2_ner-architecture.png" alt=""><figcaption><p>Рис. 5.6-2. Архитектура NER</p></figcaption></figure></div>

### Практический кейс – NER на PHP

Представим задачу.

#### Сценарий

Вы пишете CRM для юристов.

Нужно автоматически извлекать из договоров:

* имена
* даты
* компании
* суммы

Вместо регулярных выражений – используем модель NER.

#### Подход

Мы не обучаем модель.

Мы делаем inference через готовую модель – применяем уже обученную NER-модель к новому тексту. Само обучение и дообучение модели – отдельный этап, который обычно выполняется заранее.

#### Пример PHP-кода (через MITIE)

Сначала устанавливаем библиотеку:

```bash
composer require ankane/mitie-php
```

MITIE использует заранее обученную NER-модель. Для примера будем использовать любою модель, которая вам больше подходит.&#x20;

Например официальную [MITIE модель](https://github.com/mit-nlp/MITIE/releases/download/v0.4/MITIE-models-v0.2.tar.bz2) (\~436Mb).

После скачивания и распаковки модели у нас есть файл, например:

```
ner_model.dat
```

Теперь выполняем извлечение сущностей:

```php
use Mitie\Ner;

$text = "Apple signed a contract with John Smith in London for $3 million.";
$ner = new Ner(__DIR__ . "/models/ner_model.dat");

$entities = $ner->extractEntities($text);

foreach ($entities as $entity) {
    echo ($entity['text'] ?? '') . " → " . ($entity['tag'] ?? '') . "\n";
}
```

Пример вывода:

```
Apple → ORGANIZATION
John Smith → PERSON
London → LOCATION
```

#### **Что происходит внутри**

Метод:

```php
$ner->extractEntities($text);
```

получает обычную строку:

```
Apple signed a contract with John Smith in London for $3 million.
```

и превращает её в последовательность сущностей:

```php
[
    [
        "value" => "Apple",
        "tag" => "ORGANIZATION"
    ],
    [
        "value" => "John Smith",
        "tag" => "PERSON"
    ],
    [
        "value" => "London",
        "tag" => "LOCATION"
    ],
]
```

#### **Сохранение результата в структуру приложения**

Например, для CRM:

```php
$document = [
    "text" => $text,
    "entities" => []
];

foreach ($entities as $entity) {
    $document["entities"][] = [
        "type" => $entity['tag'],
        "value" => $entity['text']
    ];
}

print_r($document);
```

Результат:

```php
Array (
    [text] => Apple signed a contract with John Smith in London for $3 million.
    [entities] => Array (
        [0] => Array (
                [type] => ORGANIZATION
                [value] => Apple
            )
        [1] => Array (
                [type] => PERSON
                [value] => John Smith
            )
        [2] => Array (
                [type] => LOCATION
                [value] => London
            )
    )
)
```

#### Пример PHP-кода (через TransformersPHP)

Выполняем извлечение сущностей, используя модель `Xenova/bert-base-NER`:

```php
use Codewithkyrian\Transformers\Transformers;
use function Codewithkyrian\Transformers\Pipelines\pipeline;

$text = "Microsoft signed a contract with John Smith in London for $3 million.";
$pipeline = pipeline(task: 'token-classification', modelName: 'Xenova/bert-base-NER');
$entities = $pipeline($text);

foreach ($entities as $entity) {
    echo ($entity['word'] ?? '') . " → " . ($entity['entity'] ?? '') . "\n";
}
```

Пример вывода:

```
Microsoft → B-ORG
John → B-PER
Smith → I-PER
London → LOCATION
```

#### Почему трансформеры дали скачок качества

До трансформеров:

* LSTM учитывали контекст, но плохо масштабировались
* дальние зависимости обрабатывались тяжело

С трансформерами:

Self-attention позволяет каждому токену смотреть на все остальные:

$$
\text{Attention}(Q,K,V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}}\right)V
$$

Это означает:

Слово "Apple" может смотреть на "bought" и понять – это компания, а не фрукт.

### Где NER полезен в реальном PHP-проекте

NER – это не академическая игрушка.

Это:

* автоматическая обработка договоров
* финтех (извлечение сумм и дат)
* лог-анализ
* обработка email
* модерация контента
* построение knowledge graph

NER – это переход от текста к структуре.

### Ограничения

Важно понимать:

* модель не "понимает" текст
* она оптимизирует вероятности
* редкие сущности могут пропускаться
* доменная адаптация часто нужна

Для юридического текста – лучше дообучать модель.

### Главное понимание

NER – это первый момент в книге, где мы видим:

* AI не просто классифицирует
* AI структурирует хаос текста

С математической точки зрения: мы предсказываем последовательность меток.

С инженерной точки зрения: мы превращаем строку в данные.

И именно в этом – огромная ценность для PHP-разработчика.
