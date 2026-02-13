# Кейс 3. Автоматическое построение базы контактов из email

#### Сценарий

У вас SaaS CRM.

Пользователь загружает архив email-переписки.

Задача – извлечь:

* имена людей
* компании
* города

#### Поток обработки

1. Email → plain text
2. Текст → NER
3. Сущности → нормализация
4. Запись в таблицу contacts

#### Пример кода агрегации

```
function groupEntities(array $entities): array
{
    $result = [
        'people' => [],
        'organizations' => [],
        'locations' => []
    ];

    foreach ($entities as $entity) {
        switch ($entity['entity_group']) {
            case 'PER':
                $result['people'][] = $entity['word'];
                break;
            case 'ORG':
                $result['organizations'][] = $entity['word'];
                break;
            case 'LOC':
                $result['locations'][] = $entity['word'];
                break;
        }
    }

    return $result;
}
```

#### Что это демонстрирует

NER превращает:

```
строка → структура → база данных
```

Это и есть суть прикладного AI.
