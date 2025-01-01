# PHP with Phalcon Docker Images

Alpine-based PHP-FPM Docker images with Phalcon Framework pre-installed.

## Available Versions and Tags

### PHP 8.2 with Phalcon 5.7.0
- `8.2.19-phalcon5.7.0-alpine`: Specific version tag
- `8.2-phalcon5-alpine`: Generic version tag
- `latest`: Latest stable version

Features:
- PHP 8.2.19
- Phalcon 5.7.0
- PDO MySQL
- Imagick
- GD with WebP support
- BCMath
- Curl

### PHP 7.3 with Phalcon 3.4.5
- `7.3.33-phalcon3.4.5-alpine`: Specific version tag
- `7.3-phalcon3-alpine`: Generic version tag

Features:
- PHP 7.3.33
- Phalcon 3.4.5
- PDO MySQL
- BCMath
- JSON
- MBString
- Curl

## Usage

### PHP 8.2 with Phalcon 5.7.0
```dockerfile
FROM fvildoso/php-phalcon:8.2.19-phalcon5.7.0-alpine

# Add your application code
COPY . /var/www/html/
```

### PHP 7.3 with Phalcon 3.4.5
```dockerfile
FROM fvildoso/php-phalcon:7.3.33-phalcon3.4.5-alpine

# Add your application code
COPY . /var/www/html/
```

## Building from source

The Dockerfile for each version is available in its respective directory:
- PHP 8.2 + Phalcon 5.7.0: `php8.2-phalcon5/Dockerfile`
- PHP 7.3 + Phalcon 3.4.5: `php7.3-phalcon3/Dockerfile`

You can build the images locally using:

```bash
# For PHP 8.2 + Phalcon 5.7.0
docker build -t php-phalcon:8.2.19-phalcon5.7.0-alpine ./php8.2-phalcon5

# For PHP 7.3 + Phalcon 3.4.5
docker build -t php-phalcon:7.3.33-phalcon3.4.5-alpine ./php7.3-phalcon3
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.