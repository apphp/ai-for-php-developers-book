# Кейс 1. Предоставление доступа к инструментам разработки PHP-приложений через MCP

## **Кейс 1. Предоставление доступа к инструментам PHP-приложения через MCP**

### **Введение**

Современные AI-модели способны понимать запросы на естественном языке, анализировать информацию и помогать пользователям решать различные задачи. Однако знания самой модели не всегда достаточно. Чтобы работать с актуальными данными, выполнять операции и взаимодействовать с внешними системами, AI необходим доступ к инструментам.

Представим, что у нас есть PHP-приложение интернет-магазина. Оно уже умеет искать товары, получать информацию о заказах и обращаться к базе данных. Все эти функции доступны через существующие PHP-сервисы.

Мы хотим добавить AI-интерфейс, который позволит пользователям взаимодействовать с приложением обычным языком.

Например, пользователь может написать:

> Find me a laptop under $1000.

AI должен найти подходящие товары в нашем приложении, а не просто дать общий совет о покупке ноутбука.

Или:

> What is the status of my order #12345?

AI должен получить актуальную информацию о заказе из системы.

Для этого мы создадим MCP-сервер, который предоставит AI доступ к существующим функциям PHP-приложения.

### **Цель кейса**

Цель этого практического кейса – научиться интегрировать существующее PHP-приложение с AI через Model Context Protocol.

В процессе работы мы:

* создадим MCP-сервер на PHP;
* подключим существующие PHP-сервисы;
* предоставим бизнес-функции в виде MCP Tools;
* определим параметры и описания инструментов;
* проверим работу инструментов;
* рассмотрим взаимодействие AI с несколькими инструментами;
* изучим основные требования безопасности.

В результате мы получим сервер, который позволяет AI взаимодействовать с PHP-приложением без необходимости переписывать его бизнес-логику.

### **Сценарий**

Мы будем использовать упрощенное PHP-приложение интернет-магазина.

Приложение содержит каталог товаров и информацию о заказах.

У нас уже реализованы следующие функции:

| Функция            | Описание                           |
| ------------------ | ---------------------------------- |
| `getProduct()`     | Получает информацию о товаре по ID |
| `searchProducts()` | Ищет товары по названию и цене     |
| `getOrderStatus()` | Получает статус заказа             |

Наша задача – предоставить эти функции AI через MCP.

После интеграции пользователь сможет задавать вопросы на естественном языке, а AI будет самостоятельно определять, какой инструмент необходимо использовать.

Например:

| Пользовательский запрос            | MCP Tool           |
| ---------------------------------- | ------------------ |
| Tell me more about product 101     | `get_product`      |
| Find laptops under $1000           | `search_products`  |
| What is the status of order 12345? | `get_order_status` |

### **Архитектура решения**

В нашей архитектуре MCP-сервер будет выступать промежуточным слоем между AI и существующим PHP-приложением.

\[IMAGE PLACEHOLDER – CASE 1 ARCHITECTURE]

**Промпт для изображения:**

> Clean minimalist educational infographic, flat vector style, modern technical book illustration, white background, pastel blue, purple and green color palette, dark navy outlines, landscape 3:2 composition.
>
> Show an AI client connected to a PHP MCP Server.
>
> Inside the MCP Server show three tools:
>
> * get\_product
> * search\_products
> * get\_order\_status
>
> Connect MCP tools to PHP application services:
>
> * ProductService
> * OrderService
>
> Connect services to a Database.
>
> Show request arrows from AI to MCP tools to PHP services and database, and result arrows returning to AI.
>
> Clearly show that MCP is an integration layer and business logic remains inside the PHP application.
>
> Russian labels, no title, no photorealism.

Архитектура состоит из четырех основных компонентов:

1. **AI Client** – приложение, через которое пользователь взаимодействует с AI.
2. **MCP Server** – предоставляет AI стандартизированный интерфейс к PHP-приложению.
3. **PHP Services** – содержат бизнес-логику.
4. **Database** – хранит данные приложения.

MCP-сервер не реализует поиск товаров или работу с заказами самостоятельно. Он лишь предоставляет доступ к соответствующим сервисам.

## **Шаг 1. Подготовка окружения**

Для выполнения кейса нам понадобятся:

* PHP 8.2 или новее;
* Composer;
* Node.js и npm – для MCP Inspector;
* терминал.

Проверим установленные версии:

```
php -v
composer --version
node --version
npm --version
```

Создадим директорию проекта:

```
mkdir mcp-shop
cd mcp-shop
```

Инициализируем Composer:

```
composer init
```

Установим MCP SDK:

```
composer require mcp/sdk
composer require symfony/finder
```

MCP PHP SDK требует PHP 8.2 или новее. Symfony Finder необходим для автоматического обнаружения классов с MCP-атрибутами.

MCP PHP SDK+1

## **Шаг 2. Создание структуры проекта**

Создадим следующую структуру:

```
mcp-shop/
├── composer.json
├── server.php
├── src/
│   ├── Services/
│   │   ├── ProductService.php
│   │   └── OrderService.php
│   └── Tools/
│       └── ShopTools.php
└── vendor/
```

Каталог `Services` содержит бизнес-логику приложения.

Каталог `Tools` содержит MCP-инструменты, которые будут доступны AI.

Файл `server.php` является точкой входа MCP-сервера.

## **Шаг 3. Создание сервиса товаров**

Создадим файл:

```
src/Services/ProductService.php
```

Добавим в него следующий код:

```
<?php

namespace App\Services;

class ProductService
{
    private array $products = [
        [
            'id' => 101,
            'name' => 'Laptop Pro 14',
            'category' => 'laptop',
            'price' => 899.00,
            'available' => true,
            'description' => '14-inch laptop for everyday work.',
        ],
        [
            'id' => 102,
            'name' => 'Laptop Air 13',
            'category' => 'laptop',
            'price' => 1099.00,
            'available' => true,
            'description' => 'Lightweight laptop with a 13-inch display.',
        ],
        [
            'id' => 103,
            'name' => 'Wireless Mouse',
            'category' => 'accessory',
            'price' => 49.00,
            'available' => true,
            'description' => 'Wireless mouse for everyday use.',
        ],
        [
            'id' => 104,
            'name' => 'Mechanical Keyboard',
            'category' => 'accessory',
            'price' => 129.00,
            'available' => false,
            'description' => 'Mechanical keyboard with tactile switches.',
        ],
    ];

    public function getProduct(int $id): ?array
    {
        foreach ($this->products as $product) {
            if ($product['id'] === $id) {
                return $product;
            }
        }

        return null;
    }

    public function searchProducts(
        string $query,
        ?float $maxPrice = null
    ): array {
        $results = [];

        foreach ($this->products as $product) {
            $matchesQuery = str_contains(
                strtolower($product['name']),
                strtolower($query)
            );

            $matchesPrice = $maxPrice === null
                || $product['price'] <= $maxPrice;

            if ($matchesQuery && $matchesPrice) {
                $results[] = $product;
            }
        }

        return $results;
    }
}
```

В данном примере мы используем массив для хранения данных.

Это сделано для того, чтобы весь кейс можно было запустить без настройки базы данных.

В реальном приложении этот сервис может работать с:

* Eloquent;
* Doctrine;
* PDO;
* Repository;
* REST API;
* другими внутренними сервисами.

MCP-интеграция не зависит от конкретного способа хранения данных.

### **Проверка сервиса**

Мы можем использовать сервис напрямую из PHP:

```
require 'vendor/autoload.php';

$productService = new \App\Services\ProductService();

$product = $productService->getProduct(101);

print_r($product);
```

Результат:

```
Array
(
    [id] => 101
    [name] => Laptop Pro 14
    [category] => laptop
    [price] => 899
    [available] => 1
    [description] => 14-inch laptop for everyday work.
)
```

Таким образом, бизнес-функциональность уже существует и работает независимо от MCP.

## **Шаг 4. Создание сервиса заказов**

Создадим файл:

```
src/Services/OrderService.php
```

```
<?php

namespace App\Services;

class OrderService
{
    private array $orders = [
        12345 => [
            'id' => 12345,
            'status' => 'shipped',
            'tracking_number' => 'TRK-987654',
            'estimated_delivery' => '2026-11-25',
        ],
        12346 => [
            'id' => 12346,
            'status' => 'processing',
            'tracking_number' => null,
            'estimated_delivery' => null,
        ],
    ];

    public function getOrderStatus(int $orderId): ?array
    {
        return $this->orders[$orderId] ?? null;
    }
}
```

Сервис предоставляет информацию о заказе.

Например:

```
$orderService = new \App\Services\OrderService();

$order = $orderService->getOrderStatus(12345);

print_r($order);
```

Результат:

```
Array
(
    [id] => 12345
    [status] => shipped
    [tracking_number] => TRK-987654
    [estimated_delivery] => 2026-11-25
)
```

## **Шаг 5. Создание MCP Tools**

Теперь создадим интерфейс, через который AI сможет обращаться к нашим сервисам.

Создадим файл:

```
src/Tools/ShopTools.php
```

```
<?php

namespace App\Tools;

use App\Services\OrderService;
use App\Services\ProductService;
use Mcp\Server\Capability\Attribute\McpTool;
use Mcp\Server\Exception\ToolCallException;

class ShopTools
{
    private ProductService $productService;
    private OrderService $orderService;

    public function __construct()
    {
        $this->productService = new ProductService();
        $this->orderService = new OrderService();
    }

    /**
     * Returns detailed information about a product by its ID.
     */
    #[McpTool(
        name: 'get_product',
        description: 'Returns detailed information about a product by its ID.'
    )]
    public function getProduct(int $id): array
    {
        $product = $this->productService->getProduct($id);

        if ($product === null) {
            throw new ToolCallException(
                "Product with ID {$id} was not found."
            );
        }

        return $product;
    }

    /**
     * Searches products by name and optional maximum price.
     */
    #[McpTool(
        name: 'search_products',
        description: 'Searches products by name and optional maximum price.'
    )]
    public function searchProducts(
        string $query,
        ?float $maxPrice = null
    ): array {
        if (trim($query) === '') {
            throw new ToolCallException(
                'Search query cannot be empty.'
            );
        }

        if ($maxPrice !== null && $maxPrice < 0) {
            throw new ToolCallException(
                'Maximum price cannot be negative.'
            );
        }

        return $this->productService->searchProducts(
            $query,
            $maxPrice
        );
    }

    /**
     * Returns the current status of an order by its ID.
     */
    #[McpTool(
        name: 'get_order_status',
        description: 'Returns the current status of an order by its ID.'
    )]
    public function getOrderStatus(int $orderId): array
    {
        $order = $this->orderService->getOrderStatus($orderId);

        if ($order === null) {
            throw new ToolCallException(
                "Order {$orderId} was not found."
            );
        }

        return $order;
    }
}
```

### **Разберем инструменты**

#### `get_product`

```
#[McpTool(
    name: 'get_product',
    description: 'Returns detailed information about a product by its ID.'
)]
public function getProduct(int $id): array
```

Этот инструмент принимает ID товара и возвращает его информацию.

AI может использовать его, когда пользователь спрашивает:

* Tell me more about product 101.
* What is the price of product 103?
* Is product 101 available?

Если товар не найден, мы выбрасываем `ToolCallException`.

SDK преобразует такое исключение в результат вызова инструмента с `isError: true`, позволяя AI увидеть понятное сообщение об ошибке и при необходимости скорректировать действия.

MCP PHP SDK

#### `search_products`

```
public function searchProducts(
    string $query,
    ?float $maxPrice = null
): array
```

Инструмент принимает два параметра:

* `query` – поисковый запрос;
* `maxPrice` – максимальная цена.

Например:

```
{
  "query": "laptop",
  "maxPrice": 1000
}
```

AI может использовать этот инструмент для запросов:

* Find a laptop.
* Search for a mouse.
* Find laptops under $1000.
* Find accessories under $50.

#### `get_order_status`

```
public function getOrderStatus(int $orderId): array
```

Инструмент предоставляет информацию о заказе.

AI может использовать его, когда пользователь спрашивает:

* What is the status of order 12345?
* Has my order been shipped?
* When will order 12345 arrive?

## **Шаг 6. Создание MCP Server**

Теперь создадим точку входа сервера.

Файл:

```
server.php
```

```
<?php

require __DIR__ . '/vendor/autoload.php';

use Mcp\Server\Server;
use Mcp\Server\Transport\StdioTransport;

$server = Server::builder()
    ->setServerInfo(
        'PHP Shop MCP Server',
        '1.0.0'
    )
    ->setDiscovery(
        __DIR__,
        ['src'],
        excludeDirs: ['vendor']
    )
    ->build();

exit($server->run(new StdioTransport()));
```

Разберем конфигурацию.

#### **Информация о сервере**

```
->setServerInfo(
    'PHP Shop MCP Server',
    '1.0.0'
)
```

Мы задаем имя и версию MCP-сервера.

#### **Автоматическое обнаружение инструментов**

```
->setDiscovery(
    __DIR__,
    ['src'],
    excludeDirs: ['vendor']
)
```

SDK сканирует каталог `src` и обнаруживает классы, содержащие MCP-атрибуты.

В нашем случае он найдет:

```
#[McpTool]
```

и зарегистрирует соответствующие методы.

Автоматическое обнаружение позволяет не регистрировать каждый инструмент вручную.

#### **Запуск сервера**

```
->run(new StdioTransport())
```

Мы запускаем MCP-сервер с использованием STDIO-транспорта.

Это означает, что MCP-клиент сможет запустить наш сервер как отдельный процесс и обмениваться с ним сообщениями через стандартные потоки ввода и вывода.

Такой способ подходит для локальных AI-клиентов, IDE и desktop-приложений.

Официальный SDK поддерживает STDIO и Streamable HTTP. Для локального запуска клиент обычно запускает MCP-сервер как subprocess, а для удаленного взаимодействия используется HTTP-транспорт.

MCP PHP SDK

## **Шаг 7. Запуск MCP-сервера**

Наш проект готов.

Теперь запустим сервер:

```
php server.php
```

При использовании STDIO сервер может не выводить обычное сообщение о запуске в терминал.

Это нормально.

STDIO используется для MCP-коммуникации, поэтому обычные сообщения в `stdout` могут нарушить протокол.

Если необходимо добавить логирование, используйте `stderr` или отдельный лог-файл.

## **Шаг 8. Тестирование с MCP Inspector**

Перед подключением AI-клиента проверим наш сервер с помощью MCP Inspector.

Inspector позволяет:

* подключаться к MCP-серверу;
* просматривать список доступных Tools;
* видеть описания инструментов;
* проверять параметры;
* вручную выполнять вызовы;
* получать результаты выполнения.

Запустим:

```
npx @modelcontextprotocol/inspector php server.php
```

После запуска Inspector откроет веб-интерфейс.

\[IMAGE PLACEHOLDER – MCP Inspector testing]

**Промпт для изображения:**

> Clean minimalist educational infographic, flat vector style, modern technical book illustration, white background, pastel blue, purple and green, dark navy outlines, landscape 3:2.
>
> Show MCP Inspector testing a PHP MCP server.
>
> Left side: MCP Inspector UI with a list of tools:\
> get\_product\
> search\_products\
> get\_order\_status
>
> Right side: PHP MCP Server.
>
> Show a selected tool, its parameters and a returned JSON result.
>
> Use arrows to show a request and response.
>
> Russian labels, no title, no photorealism.

### **8.1 Проверка списка инструментов**

В Inspector мы должны увидеть три инструмента:

```
get_product
search_products
get_order_status
```

Выбрав инструмент, мы можем посмотреть его описание и параметры.

Например, `search_products`:

```
Name:
search_products

Description:
Searches products by name and optional maximum price.

Parameters:
query: string
maxPrice: number
```

SDK автоматически формирует схему параметров на основе PHP-типов и описаний. Для нашего метода:

```
public function searchProducts(
    string $query,
    ?float $maxPrice = null
): array
```

AI-клиент получает информацию о том, что `query` является обязательным параметром, а `maxPrice` – необязательным.

### **8.2 Тестирование** `search_products`

Выберем инструмент:

```
search_products
```

Передадим параметры:

```
{
  "query": "laptop",
  "maxPrice": 1000
}
```

Нажмем кнопку вызова инструмента.

MCP-сервер выполнит:

```
$this->productService->searchProducts(
    'laptop',
    1000
);
```

Результат:

```
[
  {
    "id": 101,
    "name": "Laptop Pro 14",
    "category": "laptop",
    "price": 899,
    "available": true,
    "description": "14-inch laptop for everyday work."
  }
]
```

Мы видим, что инструмент успешно вызвал бизнес-сервис и вернул данные.

### **8.3 Тестирование** `get_product`

Передадим:

```
{
  "id": 101
}
```

Результат:

```
{
  "id": 101,
  "name": "Laptop Pro 14",
  "category": "laptop",
  "price": 899,
  "available": true,
  "description": "14-inch laptop for everyday work."
}
```

Теперь попробуем запросить несуществующий товар:

```
{
  "id": 999
}
```

В результате получим ошибку инструмента:

```
Product with ID 999 was not found.
```

Это ожидаемое поведение.

### **8.4 Тестирование** `get_order_status`

Передадим:

```
{
  "orderId": 12345
}
```

Результат:

```
{
  "id": 12345,
  "status": "shipped",
  "tracking_number": "TRK-987654",
  "estimated_delivery": "2026-11-25"
}
```

Мы успешно проверили все три инструмента.

Теперь наш MCP-сервер готов к подключению AI-клиента.

## **Шаг 9. Подключение AI-клиента**

MCP-сервер сам по себе не является AI-моделью.

Он предоставляет инструменты, которые могут использовать MCP-клиенты.

Например, MCP-сервер может быть подключен к:

* AI-приложению;
* IDE;
* desktop AI client;
* собственному приложению на базе LLM API.

Клиент запускает наш сервер и получает список доступных инструментов.

Концептуально конфигурация подключения выглядит следующим образом:

```
{
  "mcpServers": {
    "php-shop": {
      "command": "php",
      "args": [
        "/absolute/path/to/mcp-shop/server.php"
      ]
    }
  }
}
```

Здесь:

* `php` – команда запуска PHP;
* `server.php` – точка входа MCP-сервера;
* `php-shop` – имя сервера, которое будет отображаться в клиенте.

Конкретный формат конфигурации зависит от AI-клиента.

Важно использовать абсолютный путь к `server.php`, чтобы клиент мог запускать сервер независимо от текущей рабочей директории.

После подключения AI-клиент сможет получить список Tools:

```
get_product
search_products
get_order_status
```

и использовать их во время общения с пользователем.

## **Шаг 10. Практические сценарии**

Теперь рассмотрим, как пользователь будет взаимодействовать с приложением.

### **Сценарий 1. Поиск товара**

Пользователь:

> Find me a laptop under $1000.

AI анализирует запрос.

Он определяет, что необходимо:

* найти товары категории laptop;
* ограничить цену $1000;
* получить результаты из приложения.

AI выбирает инструмент:

```
search_products
```

Формирует аргументы:

```
{
  "query": "laptop",
  "maxPrice": 1000
}
```

MCP-клиент передает запрос серверу.

MCP-сервер вызывает:

```
$productService->searchProducts(
    'laptop',
    1000
);
```

PHP возвращает результат:

```
[
  {
    "id": 101,
    "name": "Laptop Pro 14",
    "category": "laptop",
    "price": 899,
    "available": true
  }
]
```

AI анализирует результат и отвечает:

> I found one laptop under $1000:
>
> **Laptop Pro 14**
>
> * Price: $899
> * Availability: In stock
> * Product ID: 101
>
> Would you like more information about this laptop?

Важный момент: AI не знает актуальные товары заранее.

Он получает информацию непосредственно из нашего приложения.

### **Сценарий 2. Получение информации о товаре**

Пользователь:

> Tell me more about product 101.

AI выбирает:

```
get_product
```

и передает:

```
{
  "id": 101
}
```

PHP возвращает:

```
{
  "id": 101,
  "name": "Laptop Pro 14",
  "category": "laptop",
  "price": 899,
  "available": true,
  "description": "14-inch laptop for everyday work."
}
```

AI формирует ответ:

> Laptop Pro 14 is a 14-inch laptop designed for everyday work.
>
> The current price is $899, and it is available.
>
> Product ID: 101.

### **Сценарий 3. Проверка заказа**

Пользователь:

> What is the status of my order 12345?

AI понимает, что ему необходимо получить информацию о заказе.

Он вызывает:

```
get_order_status
```

с аргументом:

```
{
  "orderId": 12345
}
```

PHP возвращает:

```
{
  "id": 12345,
  "status": "shipped",
  "tracking_number": "TRK-987654",
  "estimated_delivery": "2026-11-25"
}
```

AI отвечает:

> Your order #12345 has been shipped.
>
> Tracking number: TRK-987654.
>
> Estimated delivery date: November 25, 2026.

AI не делает предположений о статусе заказа. Он использует реальные данные, полученные от приложения.

## **Шаг 11. Использование нескольких инструментов**

Одной из наиболее интересных возможностей AI является способность использовать несколько инструментов для решения одной задачи.

Рассмотрим запрос:

> Find a laptop under $1000 and tell me more about the best option.

Для ответа AI может выполнить несколько последовательных действий.

\[IMAGE PLACEHOLDER – Multi-tool interaction]

**Промпт для изображения:**

> Clean minimalist educational infographic, flat vector style, modern technical book illustration, white background, pastel blue, purple and green, dark navy outlines, landscape 3:2.
>
> Show an AI workflow with two sequential tool calls:
>
> 1. User asks: "Find a laptop under $1000 and tell me more about the best option."
> 2. AI calls search\_products with query="laptop", maxPrice=1000.
> 3. PHP server returns a list of matching products.
> 4. AI selects product ID 101.
> 5. AI calls get\_product with id=101.
> 6. PHP server returns product details.
> 7. AI combines the results and answers the user.
>
> Use arrows to show the sequence.\
> Clearly distinguish AI reasoning, tool calls, and PHP results.\
> Russian labels, no title.

#### **Первый вызов – поиск**

AI вызывает:

```
{
  "query": "laptop",
  "maxPrice": 1000
}
```

Получает:

```
[
  {
    "id": 101,
    "name": "Laptop Pro 14",
    "price": 899,
    "available": true
  }
]
```

#### **Второй вызов – получение подробной информации**

AI определяет, что ему нужно больше информации о найденном товаре.

Он вызывает:

```
{
  "id": 101
}
```

Инструмент `get_product` возвращает подробную информацию.

#### **Формирование ответа**

AI объединяет результаты:

> I found Laptop Pro 14 for $899.
>
> It is currently available and matches your budget.
>
> It is a 14-inch laptop designed for everyday work.
>
> Would you like to see other options?

В этом сценарии мы не написали отдельный PHP workflow, который управляет последовательностью вызовов.

AI самостоятельно выбирает инструменты и определяет, когда использовать следующий инструмент.

Это позволяет создавать гибкие AI-интерфейсы, в которых последовательность операций определяется пользовательской задачей.

## **Шаг 12. Обработка ошибок**

В реальном приложении инструменты не всегда возвращают успешный результат.

Рассмотрим несколько ситуаций.

### **Товар не найден**

Запрос:

```
{
  "id": 999
}
```

Инструмент возвращает ошибку:

```
Product with ID 999 was not found.
```

AI может ответить:

> I couldn't find a product with ID 999. Please check the product ID.

### **Заказ не найден**

Запрос:

```
{
  "orderId": 99999
}
```

Результат:

```
Order 99999 was not found.
```

AI:

> I couldn't find order #99999. Please check the order number.

### **Некорректный поисковый запрос**

Например:

```
{
  "query": "",
  "maxPrice": -100
}
```

Наш MCP Tool проверяет входные параметры:

```
if (trim($query) === '') {
    throw new ToolCallException(
        'Search query cannot be empty.'
    );
}
```

И:

```
if ($maxPrice !== null && $maxPrice < 0) {
    throw new ToolCallException(
        'Maximum price cannot be negative.'
    );
}
```

Эти проверки помогают избежать некорректных вызовов.

При этом AI может получить описание ошибки и предложить пользователю исправить запрос.

Например:

> The search query is empty. What product would you like to search for?

## **Шаг 13. Безопасность**

В нашем примере MCP-сервер предоставляет только операции чтения.

Это упрощает демонстрацию, но в реальном приложении необходимо учитывать дополнительные требования безопасности.

### **13.1 Не предоставляйте универсальный доступ к базе данных**

Не следует создавать инструмент вроде:

```
#[McpTool]
public function executeSql(string $query): array
{
    // Execute arbitrary SQL
}
```

Это предоставляет AI слишком широкие возможности.

Вместо этого создавайте конкретные инструменты:

```
search_products
get_product
get_order_status
```

Каждый инструмент должен выполнять одну определенную операцию.

### **13.2 Проверяйте доступ к данным**

Наличие инструмента:

```
get_order_status
```

не означает, что любой пользователь может получить любой заказ.

В реальном приложении необходимо проверять:

* идентичность пользователя;
* права доступа;
* принадлежность заказа пользователю;
* ограничения бизнес-логики.

Например, пользователь не должен получать информацию о чужом заказе только потому, что AI передал другой `orderId`.

### **13.3 Разделяйте операции чтения и изменения**

В дальнейшем можно добавить:

```
create_order
cancel_order
update_profile
refund_order
```

Но эти операции имеют более высокий уровень риска.

Для них может потребоваться:

* дополнительная авторизация;
* подтверждение пользователя;
* ограничение доступных параметров;
* аудит операций;
* проверка бизнес-правил.

AI может подготовить действие, но приложение должно сохранять контроль над его выполнением.

## **Шаг 14. Интеграция с существующим PHP-приложением**

В нашем примере сервисы содержат тестовые данные. В реальном проекте они могут уже существовать.

Например:

```
$productService->getProduct($id);
$productService->searchProducts($query, $maxPrice);
$orderService->getOrderStatus($orderId);
```

Если эти методы уже используются приложением, нам не нужно переписывать их для MCP.

Мы просто создаем MCP-интерфейс:

```
                    ┌──────────────┐
                    │   Web UI     │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   REST API   │
                    └──────┬───────┘
                           │
┌─────────────┐     ┌──────▼───────┐
│ MCP Server  ├────►│ PHP Services │
└─────────────┘     └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │   Database   │
                    └─────────────┘
```

Все интерфейсы используют одни и те же бизнес-сервисы.

Это позволяет добавлять AI-функциональность постепенно, без изменения существующей архитектуры приложения.

## **Шаг 15. Что можно улучшить дальше?**

Наш MCP-сервер реализует базовый сценарий, но его можно расширить.

### **Добавить новые инструменты**

Например:

```
get_product_reviews
get_product_inventory
get_recommended_products
compare_products
get_customer_orders
```

AI сможет использовать их в более сложных сценариях.

### **Добавить Resources**

Например, предоставить AI информацию о структуре каталога:

```
shop://catalog/categories
```

Или:

```
shop://policies/returns
```

Resources могут предоставлять справочную информацию, которую AI использует при формировании ответов.

### **Добавить Prompts**

Можно предоставить готовые сценарии:

```
product-comparison
customer-support
order-assistance
```

Например, Prompt `product-comparison` может содержать инструкцию:

> Compare the selected products based on price, availability, category, and specifications. Present the result in a table and explain the main differences.

### **Добавить новые бизнес-сценарии**

Например:

> Compare the two cheapest laptops under $1000.

AI может:

1. Выполнить поиск товаров.
2. Выбрать подходящие варианты.
3. Получить подробную информацию.
4. Сравнить характеристики.
5. Сформировать ответ.

И все это без создания отдельного PHP-метода для каждого пользовательского сценария.

## **Итог кейса**

В этом кейсе мы создали MCP-сервер, который предоставляет AI доступ к функциональности PHP-приложения интернет-магазина.

Мы реализовали три инструмента:

| Tool               | Назначение                    |
| ------------------ | ----------------------------- |
| `get_product`      | Получение информации о товаре |
| `search_products`  | Поиск товаров                 |
| `get_order_status` | Проверка статуса заказа       |

Мы использовали существующие PHP-сервисы и добавили MCP-слой, который делает их доступными AI.

В процессе работы мы:

1. Создали PHP-проект.
2. Установили MCP SDK.
3. Реализовали сервисы товаров и заказов.
4. Создали MCP Tools.
5. Настроили MCP Server.
6. Запустили сервер через STDIO.
7. Проверили инструменты с помощью MCP Inspector.
8. Рассмотрели взаимодействие AI с инструментами.
9. Продемонстрировали последовательное использование нескольких Tools.
10. Обсудили обработку ошибок и безопасность.

### **Главный вывод**

MCP позволяет превратить существующие функции PHP-приложения в инструменты, доступные AI.

При этом не требуется переписывать бизнес-логику или создавать отдельную систему для AI.

Мы добавляем стандартизированный интерфейс, через который AI может:

* получать актуальные данные;
* выполнять доступные операции;
* комбинировать несколько инструментов;
* решать пользовательские задачи на естественном языке.

Таким образом, MCP открывает возможность постепенно интегрировать AI в существующие PHP-приложения, сохраняя архитектуру и используя уже реализованную бизнес-функциональность.

### **Ссылки для дальнейшего изучения**

* [MCP PHP SDK – официальная документация](https://php.sdk.modelcontextprotocol.io/?utm_source=chatgpt.com)
* [Getting Started](https://php.sdk.modelcontextprotocol.io/get-started/?utm_source=chatgpt.com)
* [Tools](https://php.sdk.modelcontextprotocol.io/servers/tools/?utm_source=chatgpt.com)
* [Running an MCP Server](https://php.sdk.modelcontextprotocol.io/run/?utm_source=chatgpt.com)
* [Examples](https://php.sdk.modelcontextprotocol.io/examples/?utm_source=chatgpt.com)
* [MCP PHP SDK – GitHub](https://github.com/modelcontextprotocol/php-sdk?utm_source=chatgpt.com)

Результат: мы создали работающий MCP-сервер, который предоставляет AI доступ к функциям PHP-приложения. Это базовая архитектура, которую можно использовать для интеграции AI в существующие CRM, e-commerce, SaaS и другие PHP-системы.
