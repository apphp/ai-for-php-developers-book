# Установка напрямую

#### Введение

Ниже вы найдете подробные инструкции по подготовке и настройке среды для PHP непосредственно на вашем компьютере.

#### 1. Установка PHP 8.5

(последняя стабильная версия по состоянию на январь 2026 г.)

**Для Windows:**

1. Загрузите последнюю версию PHP 8.5 с официального сайта PHP ([https://windows.php.net/download/](https://windows.php.net/download/))
2. Распакуйте ZIP-файл в каталог (например, `C:\php`)
3. Добавьте каталог PHP в переменную среды PATH вашей системы
4. Переименуйте `php.ini-development` в `php.ini` в каталоге PHP

**Для macOS:**

```bash
brew install php@8.5
echo 'export PATH="/usr/local/opt/php@8.5/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Для Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.5 php8.5-cli php8.5-common
```

#### 2. Настройка PHP для оптимальной производительности

Отредактируйте файл `php.ini`:

```ini
memory_limit = 512M
max_execution_time = 300
error_reporting = E_ALL
display_errors = On
opcache.enable = 1
opcache.memory_consumption = 128
```

#### 3. Настройка веб-сервера

**Apache (с mod\_php):**

```bash
# Для Ubuntu/Debian
sudo apt install apache2 libapache2-mod-php8.5
sudo a2enmod php8.5
sudo systemctl restart apache2
```

**Nginx (с PHP-FPM):**

```bash
# Для Ubuntu/Debian
sudo apt install nginx php8.5-fpm
sudo systemctl start php8.5-fpm
sudo systemctl enable php8.5-fpm
```

Настройте Nginx для работы с PHP-FPM (отредактируйте `/etc/nginx/sites-available/default`):

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php8.5-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
}
```

#### 4. Установка необходимых расширений PHP для машинного обучения

```bash
sudo apt install php8.5-xml php8.5-mbstring php8.5-curl php8.5-gd php8.5-zip php8.5-mysql php8.5-bcmath
```

#### 5. Настройка Composer (менеджера пакетов)

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```

#### 6. Установка популярных библиотек машинного обучения для PHP

```bash
composer require php-ai/php-ml
composer require rubix/ml
```

#### 7. Настройка IDE

1. Установите PHPStorm или VS Code
2. Установите расширения PHP (для VS Code):

* PHP IntelliSense
* PHP Debug
* PHP Intellephense

#### 8. Настройка системы контроля версий (Git)

```bash
sudo apt install git
git config --global user.name "Ваше имя"
git config --global user.email "your.email@example.com"
```

#### 9. Настройка системы баз данных (MySQL)

Это опциональная настройка.

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```

Настройте PHP для работы с MySQL:

```bash
sudo apt install php8.5-mysql
```

#### 10. Настройка виртуальной среды

Установите и используйте встроенный сервер разработки PHP для изолированных проектов:

```bash
mkdir my_ml_project
cd my_ml_project
php -S localhost:8000
```

#### 11. Проверка настройки

Создайте файл `phpinfo.php` в корневом каталоге вашего веб-сервера со следующим содержимым:

```php
phpinfo();
```

Откройте этот файл в веб-браузере, чтобы проверить конфигурацию PHP.

#### 12. Создание простого тестового скрипта для машинного обучения

Мы создадим два тестовых скрипта, по одному для каждой библиотеки, чтобы убедиться в корректной работе PHP-ML и Rubix ML.

**Тестовый скрипт для PHP-ML**

Создайте файл с именем `php_ml_test.php` в каталоге `public`:

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
php php_ml_test.php
php rubix_ml_test.php
```

Если все настроено правильно, вы должны увидеть результат прогнозирования.
