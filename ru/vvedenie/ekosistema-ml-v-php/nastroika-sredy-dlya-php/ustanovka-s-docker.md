# Установка с Docker

#### Введение

Это руководство содержит пошаговые инструкции по настройке среды PHP 8, адаптированной для разработки машинного обучения, с использованием [Docker](https://www.docker.com/). Docker предлагает согласованную и изолированную среду, что упрощает управление зависимостями и обеспечивает воспроизводимость на разных системах. Эта настройка идеально подходит для разработчиков, желающих использовать улучшенную производительность и возможности PHP 8 для задач машинного обучения, не беспокоясь о специфических для системы конфигурациях.

#### Предварительные условия

Перед началом убедитесь, что на вашей системе установлены следующие компоненты:

1. Docker
2. Docker Compose

Инструкции по установке см. на официальном сайте Docker ([https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)) и следуйте руководству для вашей операционной системы.

#### Шаг 1: Настройка структуры проекта

Создайте новую директорию для вашего проекта и перейдите в неё:

```bash
mkdir php-ml-project
cd php-ml-project
```

#### Шаг 2: Создание Dockerfile

Создайте файл с именем `Dockerfile` в директории вашего проекта со следующим содержимым:

```dockerfile
FROM php:8.5-fpm

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    libzip-dev \
    zip \
    unzip \
    git \
    libxml2-dev \
    libcurl4-openssl-dev \
    libpng-dev \
    libonig-dev \
    && rm -rf /var/lib/apt/lists/*

# Установка расширений PHP
RUN docker-php-ext-install zip pdo_mysql bcmath xml mbstring curl gd

# Установка Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Установка рабочего каталога
WORKDIR /var/www

# Копирование содержимого существующего каталога приложения
COPY . /var/www

# Настройка PHP
RUN echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/docker-php-ram-limit.ini
RUN echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/docker-php-max-execution-time.ini

# Открыть порт 9000 и запустить сервер php-fpm
EXPOSE 9000
CMD ["php-fpm"]
```

#### Шаг 3: Создание файла Docker Compose

Создайте файл с именем `docker-compose.yml` в каталоге вашего проекта со следующим содержимым:

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/var/www
    ports:
      - "9000:9000"
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - .:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: ml_database
      MYSQL_USER: ml_user
      MYSQL_PASSWORD: ml_password
    ports:
      - "3306:3306"
```

#### Шаг 4: Создание конфигурации Nginx

Создайте файл с именем `nginx.conf` в каталоге вашего проекта со следующим содержимым: содержимое:

```nginx
server {
    listen 80;
    index index.php index.html;
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;  
    }
}
```

#### Шаг 5: Создание структуры PHP-проекта

Создайте каталог `public` и файл `index.php`:

```bash
mkdir public
echo "<?php phpinfo();" > public/index.php
```

#### Шаг 6: Создание конфигурации Composer

Создайте файл с именем `composer.json` в каталоге вашего проекта со следующим содержимым:

```json
{
    "require": {
        "php": "^8.5",
        "php-ai/php-ml": "^0.10.0",
        "rubix/ml": "^2.5.3"
    }
}
```

#### Шаг 7: Сборка и запуск контейнеров Docker

Выполните следующую команду для сборки и запуска контейнеров Docker:

```bash
docker-compose up -d --build
```

#### Шаг 8: Установка зависимостей PHP

После запуска контейнеров установите зависимости PHP:

```bash
docker-compose exec app composer install
```

#### Шаг 9: Проверка Настройки

Откройте веб-браузер и перейдите по адресу `http://localhost`. Вы должны увидеть страницу с информацией о PHP, подтверждающую правильность вашей настройки.

#### Шаг 10: Создание простого тестового скрипта для машинного обучения

Мы создадим два тестовых скрипта, по одному для каждой библиотеки, чтобы проверить правильность работы PHP-ML и Rubix ML.

#### Тестовый скрипт для PHP-ML

Создайте файл с именем `php_ml_test.php` в вашей директории `public`:

```php
require_once __DIR__ . '/../vendor/autoload.php';

use Phpml\Classification\KNearestNeighbors;

$samples = [[1, 3], [1, 4], [2, 4], [3, 1], [4, 1], [4, 2]];
$labels = ['a', 'a', 'a', 'b', 'b', 'b'];

$classifier = new KNearestNeighbors();
$classifier->train($samples, $labels);

$prediction = $classifier->predict([3, 2]);
echo "Предсказание: " . $prediction;
```

**Тестовый скрипт Rubix ML**

Создайте еще один файл с именем `rubix_ml_test.php` в вашей директории `public`:

```php
require_once __DIR__ . '/../vendor/autoload.php';

use Rubix\ML\Classifiers\KNearestNeighbors;
use Rubix\ML\Datasets\Labeled;
use Rubix\ML\Datasets\Unlabeled;

$samples = [[1, 3], [1, 4], [2, 4], [3, 1], [4, 1], [4, 2]];
$labels = ['a', 'a', 'a', 'b', 'b', 'b'];

$dataset = new Labeled($samples, $labels);
$estimator = new KNearestNeighbors(3);
$estimator->train($dataset);

$testSamples = new Unlabeled([[3, 2]]);
$prediction = $estimator->predict($testSamples);
echo "Rubix ML Prediction: " . $prediction[0] . "\n";
```

Для запуска этих скриптов используйте следующие команды:

```bash
docker-compose exec app php public/php_ml_test.php
docker-compose exec app php public/rubix_ml_test.php
```

Если все настроено правильно, вы должны увидеть результат прогнозирования.

#### Дополнительные команды Docker

Вот несколько полезных команд Docker для управления вашей средой:

* Остановка контейнеров: `docker-compose down`
* Просмотр логов контейнеров: `docker-compose logs`
* Доступ к оболочке контейнера PHP: `docker-compose exec app bash`
* Запуск скриптов PHP: `docker-compose exec app php your_script.php`

#### Заключение

Теперь у вас есть среда PHP 8 на основе Docker для разработки машинного обучения. Эта настройка включает PHP 8.5, Nginx, MySQL, PHP-ML и библиотеки Rubix ML. В этой изолированной и воспроизводимой среде вы можете начать разработку приложений машинного обучения на PHP.

Не забудьте пересобрать образ Docker, если внесете изменения в Docker file:

```bash
docker-compose up -d --build
```
