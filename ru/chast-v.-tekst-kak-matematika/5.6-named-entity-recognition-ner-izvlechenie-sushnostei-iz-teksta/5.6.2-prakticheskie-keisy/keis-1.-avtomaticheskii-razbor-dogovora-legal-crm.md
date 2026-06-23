# Кейс 1.  Автоматический разбор договора (Legal / CRM)

#### Цель кейса

В этом кейсе мы построим простой AI-пайплайн для юридической CRM.

Представим систему, в которую пользователи загружают договоры. Обычно после загрузки документа сотруднику необходимо вручную найти:

* кто является сторонами договора;
* дату подписания;
* сумму сделки;
* связанные организации;
* места и юрисдикции.

Для небольшого количества документов это возможно делать вручную. Но когда в системе находятся тысячи договоров, такой подход становится дорогим и медленным.

Задача NER – превратить обычный текст договора в структурированные данные, которые уже может использовать приложение.

После обработки документ превращается из:

```
This Agreement is made on March 12, 2025 between Apple Inc. and John Smith in London 
for the amount of $3,000,000.
```

в:

```json
{
  "organizations": [
    "Apple Inc."
  ],
  "persons": [
    "John Smith"
  ],
  "locations": [
    "London"
  ],
  "dates": [
    "March 12, 2025"
  ],
  "money": [
    "$3,000,000"
  ]
}
```

Теперь эти данные можно использовать в бизнес-логике.

#### Сценарий

Юридическая CRM получает новый договор.

Пользователь загружает документ:

```
contract_2025.pdf
```

Система выполняет следующий pipeline:

```
PDF
 |
 v
Text extraction
 |
 v
NER model
 |
 v
Entities
 |
 v
Database
 |
 v
CRM interface
```

После обработки договор автоматически получает:

* карточку компании;
* дату договора;
* сумму сделки;
* участников;
* связанные сущности.

#### Подготовка окружения

В этом примере покажем два варианта:

1. TransformersPHP – работа с моделью Transformer.
2. MITIE – локальная NER-модель через PHP.

#### Вариант 1. NER через TransformersPHP

Устанавливаем библиотеку:

```bash
composer require codewithkyrian/transformers
```

Подключаем:

```php
<?php

require 'vendor/autoload.php';

use Codewithkyrian\Transformers\Transformers;

$pipeline = Transformers::pipeline(
    'token-classification',
    'Xenova/bert-base-NER'
);
```

Теперь передаем договор в модель:

```php
$text = "
This Agreement is made on March 12, 2025 between 
Apple Inc. and John Smith.

The agreement concerns software development services.
The total contract value is $3,000,000.

The services will be provided in London, United Kingdom.
The contract becomes effective on March 12, 2025.
";


$result = $pipeline->run($text);

print_r($result);
```

Пример результата:

```php
[
    [
        "word" => "Apple",
        "entity" => "B-ORG"
    ],
    [
        "word" => "Inc.",
        "entity" => "I-ORG"
    ],
    [
        "word" => "John",
        "entity" => "B-PER"
    ],
    [
        "word" => "Smith",
        "entity" => "I-PER"
    ],
    [
        "word" => "London",
        "entity" => "B-LOC"
    ]
]
```

Здесь модель возвращает не готовые сущности, а токены с BIO-разметкой.

Например:

```
Apple → B-ORG
Inc.  → I-ORG
```

означает:

```
Apple Inc.
```

это одна организация.

#### Вариант 2. NER через MITIE (локально)

Для PHP можно использовать библиотеку:

```bash
composer require ankane/mitie-php
```

После установки у нас есть модель:

```
models/ner_model.dat
```

Теперь выполняем извлечение сущностей:

```php
<?php

require 'vendor/autoload.php';

$ner = new \Mitie\Ner(
    __DIR__ . "/models/ner_model.dat"
);


$text = "
This Agreement is made on March 12, 2025 between 
Apple Inc. and John Smith.

Apple Inc. agrees to provide software development 
services to the client.

The total amount of the agreement is $3,000,000.

The services will be delivered in London, United Kingdom.

The contract duration starts on March 12, 2025 
and expires on March 12, 2027.

The agreement was approved by Michael Brown,
Legal Director of Apple Inc.
";


$entities = $ner->extractEntities($text);


foreach ($entities as $entity) {

    echo $entity->getTag()
        . " → "
        . $entity->getValue()
        . PHP_EOL;
}
```

#### Результат работы

Пример вывода:

```
ORGANIZATION → Apple Inc.

PERSON → John Smith

LOCATION → London

MONEY → $3,000,000

DATE → March 12, 2025

DATE → March 12, 2027

PERSON → Michael Brown

ORGANIZATION → Apple Inc.
```

#### Сохранение результата в CRM

Теперь сущности можно сохранить в базу.

Например:

Таблица:

```sql
CREATE TABLE contract_entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    contract_id INT,
    entity_type VARCHAR(50),
    entity_value TEXT
);
```

PHP:

```php
$stmt = $pdo->prepare("
    INSERT INTO contract_entities
    (
        contract_id,
        entity_type,
        entity_value
    )
    VALUES (?, ?, ?)
");


foreach ($entities as $entity) {

    $stmt->execute([
        1001,
        $entity->getTag(),
        $entity->getValue()
    ]);
}
```

Теперь CRM знает:

```
Договор #1001

Компания:
Apple Inc.

Контакт:
John Smith

Сумма:
$3,000,000

Дата:
March 12, 2025

Локация:
London
```

#### Что можно построить поверх этого

После извлечения сущностей появляются новые возможности.

**Автоматическое создание карточек**

Если найдено:

```
ORGANIZATION → Apple Inc.
```

CRM может:

* найти существующую компанию;
* создать новую запись;
* связать договор.

**Проверка бизнес-правил**

Например:

```
Если сумма договора > $1,000,000
→ отправить на дополнительное согласование
```

NER становится частью workflow.

**Построение knowledge graph**

Можно построить связи:

```
John Smith
      |
      подписал
      |
Contract #1001
      |
      с компанией
      |
Apple Inc.
      |
      сумма
      |
$3,000,000
```

#### Инженерный смысл

Главная ценность NER здесь не в том, что модель “нашла слова”.

Она сделала более важное:

```
текст
 ↓
структура
 ↓
бизнес-логика
```

Обычная строка:

```
Apple Inc. signed contract with John Smith...
```

превратилась в данные:

```json
{
  "company": "Apple Inc.",
  "person": "John Smith",
  "amount": "$3,000,000"
}
```

Именно в этот момент AI начинает работать как часть backend-системы.

#### Выводы

В этом кейсе мы увидели:

* NER может быть встроен непосредственно в PHP-приложение;
* модели не заменяют бизнес-логику, а подготавливают данные для неё;
* извлечение сущностей – это мост между текстом и структурированными системами;
* один и тот же подход применяется не только к договорам, но и к email, тикетам поддержки, документам и базам знаний.

NER превращает неструктурированный текст в объект, с которым привычный PHP-код уже умеет работать.
