# Кейс 1.  Автоматический разбор договора (Legal / CRM)

#### Сценарий

Юридическая CRM получает текст договора.

Нужно автоматически извлечь:

* стороны договора
* даты
* суммы
* локации

#### Пример входного текста

```
This Agreement is made on March 12, 2025 between Apple Inc. and John Smith in London for the amount of $3,000,000.
```

***

### через MITIE (локально)

Устанавливаем:

```
composer require ankane/mitie-php
```

Загружаем модель:

```
<?php

require 'vendor/autoload.php';

$ner = new \Mitie\Ner('/path/to/ner_model.dat');

$text = "This Agreement is made on March 12, 2025 between Apple Inc. and John Smith in London for the amount of $3,000,000.";

$entities = $ner->extractEntities($text);

foreach ($entities as $entity) {
    echo $entity->getTag() . " → " . $entity->getValue() . PHP_EOL;
}
```

Пример вывода:

```
PERSON → John Smith
ORGANIZATION → Apple Inc.
LOCATION → London
DATE → March 12, 2025
MONEY → $3,000,000
```

### Инженерный смысл

После этого можно:

* сохранить в БД
* создать карточки сторон
* проверить лимиты суммы
* построить knowledge graph

Это уже автоматизация бизнеса, а не "игрушка".
