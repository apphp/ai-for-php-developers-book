# Кейс 3. AI-ассистент для SQL-аналитики (LLPhant)

LLPhant – популярная библиотека для работы с LLM в PHP.

Она поддерживает:

– агенты

– инструменты

– RAG

– embeddings

#### Цель

Создать агента, который может отвечать на вопросы к базе данных.

#### Сценарий

Пользователь спрашивает:

```
"Сколько пользователей зарегистрировалось за последние 7 дней?"
```

#### Шаги агента

1 понять, что нужен SQL

2 сгенерировать запрос

3 выполнить его

4 интерпретировать результат

#### Инструмент SQL

```
class SqlTool
{
    private PDO $pdo;

    public function run(string $query)
    {
        $stmt = $this->pdo->query($query);

        return $stmt->fetchAll();
    }
}
```

#### Пример reasoning

```
User question

↓

Generate SQL

↓

Run query

↓

Explain result
```

#### Пример SQL

```
SELECT COUNT(*)
FROM users
WHERE created_at > NOW() - INTERVAL 7 DAY
```

#### Вывод

Такой агент превращает обычную базу данных в AI-интерфейс аналитики.
