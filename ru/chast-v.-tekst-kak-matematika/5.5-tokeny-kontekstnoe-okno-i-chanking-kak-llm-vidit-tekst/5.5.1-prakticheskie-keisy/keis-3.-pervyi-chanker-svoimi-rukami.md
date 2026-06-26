---
description: Разбиваем документ на фрагменты средствами PHP.
hidden: true
---

# Кейс 3. Первый чанкер своими руками

Полностью на чистом PHP.

Например:

```php
function chunkText(
    string $text,
    int $chunkSize = 1000,
    int $overlap = 200
): array {

    $chunks = [];

    for (
        $i = 0;
        $i < strlen($text);
        $i += ($chunkSize - $overlap)
    ) {

        $chunks[] = substr(
            $text,
            $i,
            $chunkSize
        );
    }

    return $chunks;
}
```

Этот кейс хорошо объясняет:

* что такое chunk;
* что такое overlap;
* почему соседние фрагменты пересекаются.

При этом читатель понимает алгоритм ещё до знакомства с токенами.
