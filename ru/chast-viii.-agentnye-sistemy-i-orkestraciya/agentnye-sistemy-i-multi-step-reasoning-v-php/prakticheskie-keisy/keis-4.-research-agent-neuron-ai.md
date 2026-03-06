# Кейс 4. Research-агент (Neuron AI)

Neuron AI – библиотека, ориентированная на создание агентов и цепочек reasoning.

#### Цель

Создать агента, который исследует тему и делает вывод.

#### Сценарий

Запрос:

```
"Почему LLM ошибаются в математике?"
```

#### Шаги агента

1 поиск информации

2 суммаризация

3 формирование ответа

#### Подключение модели

```
use NeuronAI\Agent;
use NeuronAI\Providers\OpenAI;

$agent = new Agent(
    new OpenAI(apiKey: getenv('OPENAI_KEY'))
);
```

#### Добавление инструмента

```
$agent->tool(
    name: "search",
    description: "Search internet for information",
    callback: function($query) {

        return file_get_contents(
            "https://api.search/?q=" . urlencode($query)
        );
    }
);
```

#### Запуск агента

```
$response = $agent->run(
    "Explain why LLM fail at math"
);

echo $response;
```

#### Вывод

Neuron AI позволяет быстро создать исследовательского агента, который способен выполнять многошаговый анализ.





