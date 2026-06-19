# Installing and Configuring Transformer in PHP

### Installing TransformersPHP

TransformersPHP is a library that allows you to run modern machine learning models directly from PHP applications. Unlike many AI solutions, it does not require a separate Python service and can be used within a standard PHP project, including Laravel applications.

Official website: [https://transformers.codewithkyrian.com](https://transformers.codewithkyrian.com/)

Under the hood, the library relies on native libraries, ONNX Runtime, and PHP FFI (Foreign Function Interface), so the environment must be configured correctly.

Minimum Requirements

* PHP 8.1 or higher
* Composer
* FFI enabled
* Access to ONNX Runtime

In this chapter, we will cover two installation approaches:

1. Direct installation on a local machine or server
2. Installation inside a Docker container

For production projects, the Docker-based approach is recommended because it allows you to lock PHP versions, system libraries, and the AI runtime, ensuring a fully reproducible environment.

### Option 1. Installing TransformersPHP Directly

#### Verifying the Environment

Check your PHP version:

```bash
php -v
```

Expected output:

```
PHP 8.1.x
```

or later.

Check Composer:

```bash
composer --version
```

Verify that FFI is available:

```bash
php --ri ffi
```

or:

```bash
php -m | grep -i ffi
```

If FFI is unavailable, review your PHP configuration.

In many modern PHP distributions, FFI is already included and only needs to be enabled:

```ini
ffi.enable=1
```

After updating the configuration, restart PHP-FPM or your web server and verify again.

#### Creating a Project

Create a new directory:

```bash
mkdir transformers-demo
cd transformers-demo
```

Install the library:

```bash
composer require codewithkyrian/transformers
```

Composer will automatically create a `composer.json` file if one does not already exist.

#### Installing Native Dependencies

After installing the package, run:

```bash
./vendor/bin/transformers install
```

This command downloads and configures the platform-specific libraries required by TransformersPHP.

It is important to run this command in the same environment where the application will be executed.

For example:

* On a Linux server – run it on the server
* Inside Docker – run it inside the container
* On a local machine – run it locally

#### Verifying the Installation

Create a file:

```
index.php
```

Contents:

```php
require __DIR__ . '/vendor/autoload.php';

use Codewithkyrian\Transformers\Transformers;

echo "TransformersPHP loaded\n";
```

Run:

```bash
php index.php
```

If the message is displayed without errors, both Composer and the library are working correctly.

#### Important: The Library and the Models Are Separate Components

Installing TransformersPHP does not mean that any model has been installed.

The library provides the infrastructure for working with models, while the models themselves are typically downloaded separately.

In most cases, the first request to a model follows a flow similar to:

```
TransformersPHP
   |
Hugging Face
   |
Local Cache
```

Therefore, an internet connection is usually required for the initial model download.

In later chapters, we will examine model loading and usage in detail.

### Option 2. Installing TransformersPHP with Docker

For production development, this is generally the preferred approach.

Docker provides a consistent environment across:

* Local development
* CI/CD pipelines
* Staging servers
* Production environments

The architecture typically looks like this:

```
PHP Application
    |
TransformersPHP
    |
ONNX Runtime
    |
ML Models
```

All components run within a single container.

#### Project Structure

```
transformers-demo/
├── Dockerfile
├── docker-compose.yml
├── composer.json
└── index.php
```

#### Dockerfile

Create the following file:

```
Dockerfile
```

Contents:

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

This image:

* Installs system dependencies
* Enables the required PHP extensions
* Activates FFI
* Installs ONNX Runtime
* Adds Composer
* Installs project dependencies

#### docker-compose.yml

Create the following file:

```yaml
services:
  app:
    build:
      context: .
    container_name: transformers-demo
    volumes:
      - .:/app
```

#### Installing the Library Inside the Container

Build the image:

```bash
docker compose build
```

Install TransformersPHP:

```bash
docker compose run app composer require codewithkyrian/transformers
```

After installation, run:

```bash
docker compose run app ./vendor/bin/transformers install
```

Why is it important to execute this command inside the container?

Because the installed native libraries are platform-specific.

For example:

```
macOS
  ≠
Linux Container
```

If the installation is performed on the host machine but the application runs inside a Linux container, compatibility issues may occur.

#### Verifying the Installation

Create the following file:

```php
require __DIR__ . '/vendor/autoload.php';

echo "TransformersPHP is working\n";
```

Run:

```bash
docker compose run app php index.php
```

Expected output:

```
TransformersPHP is working
```

### Option 3. Installing TransformersPHP in an Existing Laravel Project

For Laravel, the process is almost identical.

If the project runs directly on a server:

```bash
composer require codewithkyrian/transformers
./vendor/bin/transformers install
```

If Laravel runs inside Docker:

```bash
docker compose exec app composer require codewithkyrian/transformers
docker compose exec app ./vendor/bin/transformers install
```

After installation, it is common to create a dedicated service layer:

```
app/
└── Services/
    └── AI/
        └── TransformerService.php
```

Example scaffold:

```php
namespace App\Services\AI;

class TransformerService
{
    public function analyze(string $text)
    {
        // TransformersPHP integration
    }
}
```

This architecture scales well:

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

### Common Installation Issues

#### Error: FFI extension required

Message:

```
FFI extension required
```

Check:

```bash
php --ri ffi
```

or:

```bash
php -m | grep -i ffi
```

For Docker, ensure that:

```dockerfile
RUN docker-php-ext-install ffi
```

and

```ini
ffi.enable=1
```

have been applied correctly.

#### Error: Library not found

The error may look like:

```
Library not found
```

The most common cause is a mismatch between the installation platform and the runtime platform.

Run:

```bash
./vendor/bin/transformers install
```

again in the target environment.

#### Memory Errors

Some models require a substantial amount of memory.

Check the current limit:

```bash
php -i | grep memory_limit
```

Increase it if necessary:

```ini
memory_limit=1G
```

or higher.

### Which Option Should You Choose?

For quick experimentation, a local installation is sufficient:

```
PHP + Composer
```

For Laravel projects, team development, and production systems, Docker is strongly recommended:

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

This approach makes the environment reproducible and significantly simplifies the deployment of AI functionality in PHP applications.

In the following chapters, we will use the Docker-based setup, as it most closely reflects the architecture of modern AI-powered PHP systems.
