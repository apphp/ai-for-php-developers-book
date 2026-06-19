# Установка и настройка Transformer в PHP

### Как установить TransformersPHP

TransformersPHP — это библиотека, которая позволяет запускать современные модели машинного обучения непосредственно из PHP-приложения. В отличие от многих AI-решений, она не требует отдельного Python-сервиса и может использоваться внутри обычного PHP-проекта, включая Laravel.

Официальный сайт: [https://transformers.codewithkyrian.com](https://transformers.codewithkyrian.com/)

Под капотом библиотека использует нативные библиотеки, ONNX Runtime и механизм PHP FFI (Foreign Function Interface), поэтому окружение должно быть подготовлено корректно.

Минимальные требования:

* PHP 8.1 или выше
* Composer
* включенный FFI
* доступ к ONNX Runtime

В этой главе рассмотрим два варианта установки:

1. Прямая установка на локальную машину или сервер.
2. Установка внутри Docker-контейнера.

Для реальных проектов рекомендуется Docker-подход, поскольку он позволяет зафиксировать версии PHP, системных библиотек и AI-рантайма, обеспечивая воспроизводимость окружения.

### Вариант 1. Установка TransformersPHP напрямую

#### Проверяем окружение

Проверим версию PHP:

```bash
php -v
```

Ожидаемый результат:

```
PHP 8.1.x
```

или выше.

Проверяем Composer:

```bash
composer --version
```

Проверяем доступность [FFI](https://www.php.net/manual/en/book.ffi.php):

```bash
php --ri ffi
```

или:

```bash
php -m | grep -i ffi
```

Если FFI недоступен, проверьте настройки PHP.

Во многих современных сборках PHP FFI уже встроен, и требуется лишь разрешить его использование:

```ini
ffi.enable=1
```

После изменения конфигурации перезапустите PHP-FPM или веб-сервер и повторите проверку.

#### Создаем проект

Создадим новый каталог:

```bash
mkdir transformers-demo
cd transformers-demo
```

Установим библиотеку:

```bash
composer require codewithkyrian/transformers
```

Composer автоматически создаст файл `composer.json`, если его еще нет.

#### Устанавливаем нативные зависимости

После установки пакета выполним:

```bash
./vendor/bin/transformers install
```

Команда загружает и настраивает платформозависимые библиотеки, необходимые для работы TransformersPHP.

Важно запускать эту команду именно в том окружении, где будет выполняться приложение.

Например:

* на Linux-сервере – на сервере
* внутри Docker – внутри контейнера
* на локальной машине – локально

#### Проверяем установку библиотеки

Создаем файл:

```
index.php
```

Содержимое:

```php
require __DIR__ . '/vendor/autoload.php';

use Codewithkyrian\Transformers\Transformers;

echo "TransformersPHP загружен\n";
```

Запускаем:

```bash
php index.php
```

Если сообщение выводится без ошибок, Composer и библиотека работают корректно.

#### Важно: библиотека и модели – разные вещи

Установка TransformersPHP не означает, что модель уже установлена.

Библиотека предоставляет инфраструктуру для работы с моделями, а сами модели обычно загружаются отдельно.

Чаще всего при первом обращении к модели происходит:

```
TransformersPHP
   |
Hugging Face
   |
Локальный кеш
```

Поэтому для первого запуска обычно требуется подключение к Интернету.

В последующих главах мы подробно рассмотрим загрузку и использование моделей.

### Вариант 2. Установка TransformersPHP через Docker

Для production-разработки этот вариант предпочтителен.

Docker позволяет использовать одинаковое окружение для:

* локальной разработки
* CI/CD
* staging-серверов
* production

Архитектура выглядит следующим образом:

```
PHP Application
    |
TransformersPHP
    |
ONNX Runtime
    |
ML Models
```

Все компоненты работают внутри одного контейнера.

#### Структура проекта

```
transformers-demo/
├── Dockerfile
├── docker-compose.yml
├── composer.json
└── index.php
```

#### Dockerfile

Создаем файл:

```
Dockerfile
```

Содержимое:

```dockerfile
FROM php:8.2-fpm

# ------------------------------
# Install system dependencies
# ------------------------------
RUN apt-get update && apt-get install -y \
    libzip-dev \
    zip \
    unzip \
    git \
    curl \
    libxml2-dev \
    libcurl4-openssl-dev \
    libpng-dev \
    libonig-dev \
    && rm -rf /var/lib/apt/lists/*

# ------------------------------
# Install PHP extensions
# ------------------------------
RUN docker-php-ext-install \
    zip \
    bcmath \
    xml \
    mbstring \
    curl \
    gd \
    pcntl

# ------------------------------
# Enable FFI
# ------------------------------
RUN apt-get update && apt-get install -y \
    libffi-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install ffi
RUN echo "ffi.enable=1" > /usr/local/etc/php/conf.d/ffi.ini

# ------------------------------
# Install ONNX Runtime
# ------------------------------
ENV ONNXRUNTIME_VERSION=1.17.1
RUN curl -L https://github.com/microsoft/onnxruntime/releases/download/v${ONNXRUNTIME_VERSION}/onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}.tgz \
    | tar -xz \
    && cp onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}/lib/libonnxruntime.so* /usr/lib/ \
    && ldconfig \
    && rm -rf onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}

# ------------------------------
# Install Composer
# ------------------------------
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /app

COPY composer.json composer.lock* ./

RUN composer install --no-interaction

COPY . .

CMD ["php", "index.php"]
```

В этом образе:

* устанавливаются системные зависимости
* подключаются необходимые PHP-расширения
* включается FFI
* устанавливается ONNX Runtime
* добавляется Composer
* выполняется установка зависимостей проекта

#### docker-compose.yml

Создаем файл:

```yaml
services:
  app:
    build:
      context: .
    container_name: transformers-demo
    volumes:
      - .:/app
```

#### Установка библиотеки внутри контейнера

Собираем образ:

```bash
docker compose build
```

Устанавливаем TransformersPHP:

```bash
docker compose run app composer require codewithkyrian/transformers
```

После установки выполняем:

```bash
docker compose run app ./vendor/bin/transformers install
```

Почему важно выполнять команду внутри контейнера?

Потому что устанавливаемые библиотеки зависят от платформы.

Например:

```
MacOS
  ≠
Linux Container
```

Если выполнить установку на хостовой машине, а запускать приложение внутри Linux-контейнера, возможны ошибки совместимости.

#### Проверяем работу

Создаем файл:

```php
require __DIR__ . '/vendor/autoload.php';

echo "TransformersPHP работает\n";
```

Запускаем:

```bash
docker compose run app php index.php
```

Ожидаемый результат:

```
TransformersPHP работает
```

### Вариант 3. Установка в существующий Laravel-проект

Для Laravel процедура практически не отличается.

Если проект работает напрямую на сервере:

```bash
composer require codewithkyrian/transformers
./vendor/bin/transformers install
```

Если Laravel работает внутри Docker:

```bash
docker compose exec app composer require codewithkyrian/transformers
docker compose exec app ./vendor/bin/transformers install
```

После установки обычно создают отдельный сервисный слой:

```
app/
└── Services/
    └── AI/
        └── TransformerService.php
```

Пример заготовки:

```php
namespace App\Services\AI;

class TransformerService
{
    public function analyze(string $text)
    {
        // Работа с TransformersPHP
    }
}
```

Такой подход хорошо масштабируется:

```
Laravel
    |
AI Service Layer
    |
TransformersPHP
    |
ONNX Runtime
    |
ML Models
```

### Типичные проблемы установки

#### Ошибка: FFI extension required

Сообщение:

```
FFI extension required
```

Проверьте:

```bash
php --ri ffi
```

или:

```bash
php -m | grep -i ffi
```

Для Docker убедитесь, что:

```dockerfile
RUN docker-php-ext-install ffi
```

и

```ini
ffi.enable=1
```

применены корректно.

#### Ошибка: Library not found

Сообщение может выглядеть так:

```
Library not found
```

Причина обычно заключается в несовпадении платформы установки и платформы запуска.

Повторите:

```bash
./vendor/bin/transformers install
```

в текущем окружении.

#### Ошибка памяти

Некоторые модели требуют значительный объем памяти.

Проверяем ограничение:

```bash
php -i | grep memory_limit
```

При необходимости увеличиваем:

```ini
memory_limit=1G
```

или выше.

### Какой вариант выбрать

Для быстрого знакомства с библиотекой достаточно локальной установки:

```
PHP + Composer
```

Для Laravel-проектов, командной разработки и production-систем рекомендуется Docker:

```
Docker
   |
PHP
   |
TransformersPHP
   |
ONNX Runtime
   |
ML Models
```

Такой подход делает окружение воспроизводимым и значительно упрощает развертывание AI-функциональности в PHP-приложениях.

В следующих главах мы будем использовать Docker-вариант, поскольку он наиболее близок к реальной архитектуре современных AI-систем на PHP.
