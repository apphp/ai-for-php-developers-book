# Кейс 2. Финтех – автоматический анализ транзакционных сообщений

#### Сценарий

Пользователь загружает email:

```
Transfer of $8500 was sent to Michael Brown in New York on January 5.
```

Нужно выделить:

* сумму
* получателя
* локацию
* дату

### Чистый PHP + REST API

```
function analyzeTransaction(string $text): array
{
    $payload = json_encode(["inputs" => $text]);

    $ch = curl_init("https://api.example.com/ner");
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $payload);
    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        "Content-Type: application/json",
        "Authorization: Bearer TOKEN"
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    return json_decode($response, true);
}

$text = "Transfer of $8500 was sent to Michael Brown in New York on January 5.";
$result = analyzeTransaction($text);

print_r($result);
```

### Почему это лучше регулярок

Регулярки:

* ломаются на вариациях формата
* не понимают контекст

NER:

* работает с “8500 dollars”, “$8.5k”, “eight thousand five hundred”
* понимает структуру предложения
