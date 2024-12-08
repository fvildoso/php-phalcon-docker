# PHP 8.2 with Phalcon Docker Image

Alpine-based PHP-FPM 8.2 Docker image with Phalcon Framework 5.7.0 pre-installed.

## Tags
- `8.2.19-alpine`: PHP 8.2.19 with Phalcon 5.7.0 on Alpine 3.18

## Features
- PHP 8.2.19
- Phalcon 5.7.0
- PDO MySQL
- Imagick
- GD with WebP support
- BCMath
- Curl

## Usage

```dockerfile
FROM fvildoso/php-phalcon:8.2.19-alpine

# Add your application code
COPY . /var/www/html/