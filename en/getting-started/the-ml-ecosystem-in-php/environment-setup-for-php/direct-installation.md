# Direct installation

**Introduction**

Below you will find detailed instructions for preparing and configuring a PHP environment directly on your computer.

**1. Installing PHP 8.5**

(latest stable version as of January 2026)

**For Windows:**

1. Download the latest version of PHP 8.5 from the official PHP website ([https://windows.php.net/download/](https://windows.php.net/download/))
2. Extract the ZIP file to a directory (for example, `C:\php`)
3. Add the PHP directory to your system PATH environment variable
4. Rename `php.ini-development` to `php.ini` in the PHP directory

**For macOS:**

```bash
brew install php@8.5
echo 'export PATH="/usr/local/opt/php@8.5/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**For Linux (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.5 php8.5-cli php8.5-common
```

**2. Configuring PHP for Optimal Performance**

Edit the `php.ini` file:

```ini
memory_limit = 512M
max_execution_time = 300
error_reporting = E_ALL
display_errors = On
opcache.enable = 1
opcache.memory_consumption = 128
```

**3. Web Server Configuration**

**Apache (with mod\_php):**

```bash
# For Ubuntu/Debian
sudo apt install apache2 libapache2-mod-php8.5
sudo a2enmod php8.5
sudo systemctl restart apache2
```

**Nginx (with PHP-FPM):**

```bash
# For Ubuntu/Debian
sudo apt install nginx php8.5-fpm
sudo systemctl start php8.5-fpm
sudo systemctl enable php8.5-fpm
```

Configure Nginx to work with PHP-FPM (edit `/etc/nginx/sites-available/default`):

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php8.5-fpm.sock;
    fastcgi_index index.php;
    include fastcgi_params;
}
```

**4. Installing Required PHP Extensions for Machine Learning**

```bash
sudo apt install php8.5-xml php8.5-mbstring php8.5-curl php8.5-gd php8.5-zip php8.5-mysql php8.5-bcmath
```

**5. Setting Up Composer (Package Manager)**

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```

**6. Installing Popular Machine Learning Libraries for PHP**

```bash
composer require php-ai/php-ml
composer require rubix/ml
```

**7. IDE Setup**

1. Install PHPStorm or VS Code
2. Install PHP extensions (for VS Code):

* PHP IntelliSense
* PHP Debug
* PHP Intellephense

**8. Version Control System Setup (Git)**

```bash
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**9. Database System Setup (MySQL)**

This is an optional setting.

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```

Configure PHP to work with MySQL:

```bash
sudo apt install php8.5-mysql
```

**10. Virtual Environment Setup**

Install and use the built-in PHP development server for isolated projects:

```bash
mkdir my_ml_project
cd my_ml_project
php -S localhost:8000
```

**11. Configuration Verification**

Create a file named `phpinfo.php` in the root directory of your web server with the following content:

```php
phpinfo();
```

Open this file in a web browser to verify the PHP configuration.

**12. Creating a Simple Machine Learning Test Script**

We will create two test scripts, one for each library, to ensure that PHP-ML and Rubix ML are working correctly.

**Test Script for PHP-ML**

Create a file named `php_ml_test.php` in the `public` directory:

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
php php_ml_test.php
php rubix_ml_test.php
```

If everything is configured correctly, you should see the prediction output.
