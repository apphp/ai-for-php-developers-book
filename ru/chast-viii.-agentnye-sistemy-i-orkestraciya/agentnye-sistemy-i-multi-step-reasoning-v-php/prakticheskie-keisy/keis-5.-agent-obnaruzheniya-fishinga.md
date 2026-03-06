# Кейс 5. Агент обнаружения фишинга

#### Цель

Создать агента, который анализирует письмо и определяет признаки фишинга.

#### Шаги reasoning

```
1 extract links
2 check domain reputation
3 detect suspicious wording
4 score risk
```

#### Инструменты

```
link_extractor
domain_reputation
phishing_classifier
```

#### Пример инструмента

```
class LinkExtractor
{
    public function run(string $email)
    {
        preg_match_all('/https?:\/\/\S+/', $email, $links);

        return $links[0];
    }
}
```

#### Финальная оценка

Агент может вернуть:

```
Risk score: 0.82

Reasons:
- suspicious domain
- urgent language
- external links
```















