# Кейс 4. Подготовка данных для RAG

Это особенно хорошо связывает главу с будущей RAG-главой.

#### Сценарий

У вас база из 10 000 документов.

Вы хотите:

* извлекать все компании
* строить индекс
* быстро находить документы по сущности

#### Архитектурная идея

1. Документ
2. NER
3. Сущности сохраняются в отдельную таблицу
4. При запросе – сначала фильтрация по сущностям
5. Потом – RAG

#### Таблица в БД

```
CREATE TABLE document_entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    document_id INT,
    entity_type VARCHAR(50),
    entity_value VARCHAR(255)
);
```

#### PHP вставка

```
$stmt = $pdo->prepare("
    INSERT INTO document_entities (document_id, entity_type, entity_value)
    VALUES (?, ?, ?)
");

foreach ($entities as $entity) {
    $stmt->execute([
        $documentId,
        $entity['entity_group'],
        $entity['word']
    ]);
}
```

#### Это уже системное мышление

NER перестаёт быть NLP-задачей.

Он становится частью архитектуры.
