# Кейс 1. AI-калькулятор как инструмент агента (чистый PHP)

#### Простая агентная система на PHP

Начнём с минимального агента.

#### Цель

Показать базовую архитектуру агентной системы:

– LLM планирует действие

– система выбирает инструмент

– инструмент выполняет задачу

#### Сценарий

Пользователь задаёт вопрос:

"Сколько будет (25 \* 8) + 120 ?"

Агент должен:

1 определить, что нужна арифметика

2 вызвать инструмент calculator

3 вернуть результат

#### Архитектура

User\
↓\
LLM planner\
↓\
Tool selection\
↓\
Calculator\
↓\
Answer





#### Интерфейс инструмента

```php
interface Tool {
    public function name(): string;
    public function execute(array $input): string;
}
```

#### Пример инструмента

```php
class CalculatorTool implements Tool {
    public function name(): string {
        return "calculator";
    }

    public function execute(array $input): string {
        $expr = $input["expression"];
        $result = eval("return $expr;");
        return (string)$result;
    }
}
```

### Планирование через LLM

LLM должна решить:

* какой инструмент использовать
* какие параметры передать

Например:

```php
User: 2 + 5 * 10

LLM plan:

{
  "tool": "calculator",
  "input": {
     "expression": "2 + 5 * 10"
  }
}
```

#### PHP вызов LLM

```php
function askLLM($prompt) {
    $apiKey = getenv("OPENAI_KEY");

    $data = [
        "model" => "gpt-4o-mini",
        "messages" => [
            ["role" => "user", "content" => $prompt]
        ]
    ];

    $ch = curl_init("https://api.openai.com/v1/chat/completions");

    curl_setopt($ch, CURLOPT_HTTPHEADER, [
        "Authorization: Bearer $apiKey",
        "Content-Type: application/json"
    ]);

    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

    $response = curl_exec($ch);

    return json_decode($response, true);
}
```

### Контроллер агента

Контроллер управляет циклом.

```php
class Agent {
    private $tools = [];

    public function addTool(Tool $tool) {
        $this->tools[$tool->name()] = $tool;
    }

    public function run($task) {
        $prompt = "You are an AI agent. Decide which tool to use. Task: $task. Return JSON:
{
 'tool': '...',
 'input': {...}
};"

        $response = askLLM($prompt);
        $plan = json_decode(
            $response["choices"][0]["message"]["content"],
            true
        );
        $toolName = $plan["tool"];
        $tool = $this->tools[$toolName];

        return $tool->execute($plan["input"]);
    }
}
```

#### Использование

```php
$agent = new Agent();
$agent->addTool(new CalculatorTool());
$result = $agent->run("What is 5 * 7 + 3 ?");

echo $result;
```

#### Вывод

Даже самый простой агент показывает ключевую идею:

LLM не выполняет задачу сама,

она решает, какой инструмент использовать.
