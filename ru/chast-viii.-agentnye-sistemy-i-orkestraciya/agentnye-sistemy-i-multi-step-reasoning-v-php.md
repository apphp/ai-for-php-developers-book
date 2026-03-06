---
description: 'LLM как управляемая система: планирование, инструменты и контроль.'
---

# Агентные системы и multi-step reasoning в PHP

Когда разработчики впервые сталкиваются с LLM, они обычно воспринимают модель как "умный автокомплит". Мы задаём вопрос – модель генерирует ответ. Однако такой способ использования раскрывает лишь небольшую часть потенциала современных моделей.

На практике наиболее мощные системы строятся иначе. LLM используется не как финальный источник ответа, а как компонент внутри управляемой системы, где:

* модель планирует действия
* система вызывает инструменты
* промежуточные результаты проверяются
* процесс повторяется до достижения цели

Такой подход называется агентной архитектурой.

В этой главе мы разберём:

* что такое агентные системы
* что такое multi-step reasoning
* как устроено планирование действий
* как подключать инструменты
* как контролировать выполнение

Ну а затем мы реализуем это на PHP.

### Почему одного запроса к LLM недостаточно

Представим задачу.

Пользователь спрашивает:

> "Сколько будет средняя зарплата PHP-разработчика в Германии, если учитывать последние данные?"

Чтобы ответить на этот вопрос, система должна:

1. Найти свежие данные по зарплатам
2. Отфильтровать по PHP
3. Посчитать среднее
4. Сформировать ответ

LLM не имеет доступа к интернету и вычислениям, если её специально этому не научить.

Поэтому система должна выполнить цепочку действий:

```
Запрос пользователя
      ↓
Планирование шагов
      ↓
Вызов инструментов
      ↓
Получение данных
      ↓
Анализ
      ↓
Финальный ответ
```

Так работает агентная система.

### Что такое агент

Агент – это система, которая может:

1. понимать цель
2. планировать действия
3. использовать инструменты
4. оценивать результат
5. повторять процесс

Формально можно представить агента как функцию:

$$
a_t = \pi(s_t)
$$

где

* $$s_t$$ – состояние системы
* $$a_t$$ – действие
* $$\pi$$ – стратегия агента

После действия состояние меняется:

$$
s_{t+1} = f(s_t, a_t)
$$

LLM выступает в роли политики принятия решений.

### Multi-step reasoning

Большинство сложных задач нельзя решить за один шаг.

Они требуют последовательного рассуждения. Рассмотрим следующий пример.

Запрос:

> "Какая компания дороже – Apple или Microsoft?"

Шаги:

1. Получить капитализацию Apple
2. Получить капитализацию Microsoft
3. Сравнить

Формально процесс рассуждения (reasoning) можно представить как последовательность состояний:

$$
S = (s_0, s_1, s_2, ..., s_n)
$$

где каждое состояние зависит от предыдущего.

#### Внутренний цикл агента

```php
while (!isGoalReached()) {
    observe_state();
    plan_next_step();
    use_tool();
    update_state();
}
```

#### Плейсхолдер для картинки

\[Картинка: цикл работы агента]

Prompt для генерации изображения:

```
Diagram of an AI agent loop:
User request → LLM planning → Tool usage → Observation → Next step → Final answer.
Minimalist technical diagram, white background.
```

### Архитектура агентной системы

Типичная архитектура выглядит так:

```
User
 ↓
Controller
 ↓
LLM planner
 ↓
Tool selection
 ↓
Tool execution
 ↓
Memory update
 ↓
Next step
```

Компоненты:

**1. Планировщик (Planner)**

LLM решает:

* что делать дальше
* какой инструмент использовать

**2. Инструменты (Tools)**

Функции системы:

* поиск
* API
* базы данных
* вычисления

**3. Память (Memory)**

Контекст предыдущих шагов.

**4. Контроллер**

Логика цикла выполнения.

**5. Инструменты**

Инструмент – это функция, которую агент может вызвать.

Примеры:

```
search(query)
get_weather(city)
run_sql(query)
calculate(expression)
```

Формально инструмент:

$$
tool: X \rightarrow Y
$$

### Простая агентная система на PHP

Начнём с минимального агента.

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
        $prompt = "You are an AI agent. Decide which tool to use. Task: $task";
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

### Память агента

Для multi-step reasoning нужна память.

Простейшая память – список сообщений.

```
memory = [
  user_question
  step_1
  observation_1
  step_2
]
```

#### PHP реализация

```php
class Memory {
    private $history = [];

    public function add($message) {
        $this->history[] = $message;
    }

    public function context() {
        return implode("\n", $this->history);
    }
}
```

### Планирование нескольких шагов

Агент может выполнять несколько действий.

Пример:

```
Goal: Найти среднюю зарплату

Step 1:
search salary data

Step 2:
extract numbers

Step 3:
calculate mean
```

#### Цикл агента

```php
for ($i = 0; $i < 5; $i++) {
    $plan = $planner->nextStep($memory);

    if ($plan["tool"] == "finish") {
        break;
    }

    $result = $tool->execute($plan["input"]);

    $memory->add($result);
}
```

#### Плейсхолдер для картинки

\[Картинка: multi-step reasoning]

Prompt:

```
Diagram showing multi-step reasoning in an AI agent:
step1 reasoning → tool → observation
step2 reasoning → tool → observation
step3 final answer
clean technical illustration
```

### Контроль и безопасность

Без контроля агент может:

* зациклиться
* вызвать опасный инструмент
* генерировать мусор

Поэтому система должна иметь ограничения.

#### Ограничение шагов

```
max_steps = 10
```

#### Ограничение инструментов

Белый список:

```
allowed_tools = [
  search
  calculator
  database
]
```

#### Валидация аргументов

```php
if (!is_numeric($input["expression"])) {
    throw new Exception("Invalid input");
}
```

### Evaluator – критик агента

В более сложных системах есть критик.

Он проверяет:

* правильность шага
* достижение цели

Формально:

$$
score = V(s_t)
$$

где $$V$$ – функция оценки.

LLM может выступать и в роли критика.

## Self-Reflection

Современные агентные системы используют саморефлексию.

Модель анализирует собственные ошибки.

```
Step result: incorrect

Reflection:
I used wrong tool.
Next step: use calculator.
```

### Почему агентные системы – это будущее LLM

Большинство современных AI-систем уже используют агентную архитектуру.

Причины:

1. LLM плохо считает
2. LLM не имеет данных
3. LLM делает ошибки

Агентная архитектура решает это:

* инструменты делают вычисления
* базы дают данные
* контроллер следит за процессом

В итоге LLM становится оркестратором интеллекта, а не источником истины.

### Итог

LLM сама по себе – это генератор текста.

Но внутри агентной архитектуры она превращается в универсальный планировщик действий.

Агентная система включает:

* планирование
* инструменты
* память
* контроль
* оценку

Такая архитектура позволяет решать сложные задачи через multi-step reasoning, объединяя сильные стороны LLM и традиционного программирования.

Для PHP-разработчиков это особенно важно: большая часть логики агента реализуется обычным кодом, а LLM используется только там, где нужна гибкость и семантическое понимание.

Именно поэтому современная разработка AI-систем всё чаще выглядит не как вызов модели, а как проектирование управляемой интеллектуальной системы.
