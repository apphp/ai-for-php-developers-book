# Кейс 5. Дообучение NER-модели (на примере MITIE)

Когда нужно дообучение

Предобученные модели хорошо находят:

* PERSON
* LOCATION
* ORGANIZATION

Но плохо работают с доменными сущностями:

* CONTRACT\_ID
* INVOICE\_NUMBER
* PRODUCT\_CODE
* CRYPTO\_WALLET

Если ваш бизнес-домен специфичен – модель нужно дообучить.

#### Формат обучающих данных

MITIE использует формат с разметкой сущностей.

Пример тренировочного файла:

```
John Smith works at Apple Inc.
(0, 10) PERSON
(20, 30) ORGANIZATION

Invoice #A-4456 was issued on March 5.
(8, 14) INVOICE_NUMBER
(29, 36) DATE
```

Где:

* числа – позиции символов в строке
* тип – метка сущности

#### Создание обучающего датасета

Минимальный пример training.txt:

```
Contract #C-2025-001 was signed by Michael Brown in Berlin.
(9, 21) CONTRACT_ID
(36, 49) PERSON
(53, 59) LOCATION
```

Важно: позиции должны точно совпадать с индексами строки.

#### Обучение модели через MITIE CLI

MITIE обучается через командную строку.

Пример:

```
mitie_ner_trainer training.txt
```

Или с указанием модели эмбеддингов:

```
mitie_ner_trainer \
  -r total_word_feature_extractor.dat \
  training.txt \
  new_ner_model.dat
```

Где:

* total\_word\_feature\_extractor.dat – предобученные эмбеддинги
* training.txt – ваш размеченный корпус
* new\_ner\_model.dat – выходная модель

#### Использование дообученной модели в PHP

```
<?php

require 'vendor/autoload.php';

$ner = new \Mitie\Ner('/path/to/new_ner_model.dat');

$text = "Contract #C-2025-001 was signed by Michael Brown in Berlin.";

$entities = $ner->extractEntities($text);

foreach ($entities as $entity) {
    echo $entity->getTag() . " → " . $entity->getValue() . PHP_EOL;
}
```

Теперь модель умеет находить:

```
CONTRACT_ID → C-2025-001
PERSON → Michael Brown
LOCATION → Berlin
```

#### Что происходит математически

Во время обучения MITIE:

1. Текст токенизируется
2. Для каждого токена строится вектор признаков
3. Обучается линейный классификатор

Формально:

h\_i = \text{features}(x\_i)<br>

\text{score} = W h\_i + b



Оптимизация – минимизация ошибки классификации.

Это не глубокая нейросеть.

Это структурированный линейный классификатор поверх эмбеддингов.

#### Минимальная оценка качества

Для NER обычно используют:

* Precision
* Recall
* F1-score<br>

Precision = \frac{TP}{TP + FP}

Recall = \frac{TP}{TP + FN}

F1 = 2 \cdot \frac{Precision \cdot Recall}{Precision + Recall}

Важно: в NER ошибка в границе сущности – это ошибка целиком.

Если модель предсказала:

```
Michael
```

вместо:

```
Michael Brown
```

Это считается неправильной сущностью.

#### Иллюстрация процесса обучения

\[PLACEHOLDER\_IMAGE\_FINE\_TUNING]

Промпт для изображения:

> Create a simple diagram illustrating NER fine-tuning. Show: labeled text with entity spans → feature extraction → training process → updated NER model. Minimal technical style, white background, arrows between blocks, suitable for programming book.

#### Что если использовать Transformers

Если вы дообучаете BERT-подобную модель:

1. Загружаете предобученную модель
2. Добавляете classification head
3. Обучаете на размеченном корпусе

Формально:

h\_i = \text{Transformer}(x)

P(y\_i) = \text{softmax}(W h\_i)

Fine-tuning – это обновление весов трансформера и классификатора.

Обычно это делается в Python (HuggingFace), а PHP используется только для inference.

#### Важный инженерный вывод

Дообучение нужно когда:

* домен специфичен
* данные нестандартны
* базовая модель пропускает сущности

Но:

* разметка данных – дорогая
* нужна валидация
* важен контроль качества

NER – это не только модель.

Это pipeline:

1. Разметка
2. Обучение
3. Оценка
4. Интеграция
