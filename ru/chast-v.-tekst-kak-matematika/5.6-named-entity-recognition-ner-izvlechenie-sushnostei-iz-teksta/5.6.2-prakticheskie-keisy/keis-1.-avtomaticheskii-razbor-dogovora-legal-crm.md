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



#### **USE**

use transformers-php<br>

[https://transformers.codewithkyrian.com/token-classification](https://transformers.codewithkyrian.com/token-classification)

```
require 'vendor/autoload.php';

use Codewithkyrian\Transformers\Transformers;

$pipeline = Transformers::pipeline('token-classification', 'Xenova/bert-base-NER');

// Perform NER on a sentence
$result = $pipeline->run("Apple opened a new office in N-sk.");

print_r($result);
```

<br>



###

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
    echo $entity->getTag() . " → " . $entity->getValue() . "\n;
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
