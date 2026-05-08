# Installation with Docker

**Introduction**

This guide provides step-by-step instructions for setting up a PHP 8 environment tailored for machine learning development using Docker. Docker offers a consistent and isolated environment, simplifying dependency management and ensuring reproducibility across different systems. This setup is ideal for developers who want to leverage the improved performance and capabilities of PHP 8 for machine learning tasks without worrying about system-specific configurations.

**Prerequisites**

Before you begin, make sure the following components are installed on your system:

1. Docker
2. Docker Compose

For installation instructions, visit the official Docker website ([https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)) and follow the guide for your operating system.

**Step 1: Project Structure Setup**

Create a new directory for your project and navigate into it:

```bash
mkdir php-ml-project
cd php-ml-project
```

**Step 2: Creating the Dockerfile**

Create a file named `Dockerfile` in your project directory with the following contents:

```dockerfile
FROM php:8.5-fpm

# Install system dependencies
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

# Install PHP extensions
RUN docker-php-ext-install zip pdo_mysql bcmath xml mbstring curl gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy existing application directory contents
COPY . /var/www

# PHP configuration
RUN echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/docker-php-ram-limit.ini
RUN echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/docker-php-max-execution-time.ini

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

**Step 3: Creating the Docker Compose File**

Create a file named `docker-compose.yml` in your project directory with the following contents:

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

**Step 4: Creating the Nginx Configuration**

Create a file named `nginx.conf` in your project directory with the following contents:

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

**Step 5: Creating the PHP Project Structure**

Create the `public` directory and the `index.php` file:

```bash
mkdir public
echo "<?php phpinfo();" > public/index.php
```

**Step 6: Creating the Composer Configuration**

Create a file named `composer.json` in your project directory with the following contents:

```json
{
    "require": {
        "php": "^8.5",
        "php-ai/php-ml": "^0.10.0",
        "rubix/ml": "^2.5.3"
    }
}
```

**Step 7: Building and Running Docker Containers**

Run the following command to build and start the Docker containers:

```bash
docker-compose up -d --build
```

**Step 8: Installing PHP Dependencies**

After the containers are running, install the PHP dependencies:

```bash
docker-compose exec app composer install
```

**Step 9: Verifying the Setup**

Open a web browser and go to `http://localhost`. You should see the PHP information page, confirming that your setup is correct.

**Step 10: Creating a Simple Machine Learning Test Script**

We will create two test scripts, one for each library, to verify that PHP-ML and Rubix ML are working correctly.

**Test Script for PHP-ML**

Create a file named `php_ml_test.php` in your `public` directory:

```php
require_once __DIR__ . '/../vendor/autoload.php';

use Phpml\Classification\KNearestNeighbors;

$samples = [[1, 3], [1, 4], [2, 4], [3, 1], [4, 1], [4, 2]];
$labels = ['a', 'a', 'a', 'b', 'b', 'b'];

$classifier = new KNearestNeighbors();
$classifier->train($samples, $labels);

$prediction = $classifier->predict([3, 2]);
echo "Prediction: " . $prediction;
```

**Rubix ML Test Script**

Create another file named `rubix_ml_test.php` in your `public` directory:

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

To run these scripts, use the following commands:

```bash
docker-compose exec app php public/php_ml_test.php
docker-compose exec app php public/rubix_ml_test.php
```

If everything is set up correctly, you should see the prediction output.

**Additional Docker Commands**

Here are some useful Docker commands for managing your environment:

* Stop containers: `docker-compose down`
* View container logs: `docker-compose logs`
* Access the PHP container shell: `docker-compose exec app bash`
* Run PHP scripts: `docker-compose exec app php your_script.php`

**Conclusion**

You now have a Docker-based PHP 8 environment for machine learning development. This setup includes PHP 8.5, Nginx, MySQL, and the PHP-ML and Rubix ML libraries. Within this isolated and reproducible environment, you can begin developing machine learning applications in PHP.

Do not forget to rebuild the Docker image if you make changes to the Dockerfile:

```bash
docker-compose up -d --build
```
