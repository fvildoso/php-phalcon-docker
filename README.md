# PHP with Phalcon Docker Images

Multi-arch PHP-FPM Docker images with the Phalcon Framework pre-installed.

Supported architectures: linux/amd64 and linux/arm64 (multi-arch images).

Base images:
- PHP 8.4 images are Debian-based.
- PHP 8.2 and 7.3 images are Alpine-based.

## Available Versions and Tags

### PHP 8.4 with Phalcon 5.9.4 (Debian-based)
- `8.4-phalcon5.9.4`: Specific version tag
- `8.4-phalcon5.9`: Generic moving tag

Features:
- PHP 8.4.x
- Phalcon 5.9.4
- PDO MySQL
- Imagick
- GD
- BCMath
- Curl

### PHP 8.2 with Phalcon 5.7.0 (Alpine-based)
- `8.2.19-phalcon5.7.0-alpine`: Specific version tag
- `8.2-phalcon5-alpine`: Generic version tag
- `latest`: Latest stable tag (currently points to PHP 8.2)

Features:
- PHP 8.2.19
- Phalcon 5.7.0
- PDO MySQL
- Imagick
- GD with WebP support
- BCMath
- Curl

### PHP 7.3 with Phalcon 3.4.5 (Alpine-based)
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

### PHP 8.4 with Phalcon 5.9.4
```dockerfile
FROM fvildoso/php-phalcon:8.4-phalcon5.9.4

# Add your application code
COPY . /var/www/html/
```

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
- PHP 8.4 + Phalcon 5.9.4: `php8.4-phalcon5.9/Dockerfile`
- PHP 8.2 + Phalcon 5.7.0: `php8.2-phalcon5/Dockerfile`
- PHP 7.3 + Phalcon 3.4.5: `php7.3-phalcon3/Dockerfile`

You can build the images locally using:

```bash
# For PHP 8.4 + Phalcon 5.9.4 (Debian-based)
docker build -t fvildoso/php-phalcon:8.4-phalcon5.9.4 ./php8.4-phalcon5.9

docker tag fvildoso/php-phalcon:8.4-phalcon5.9.4 fvildoso/php-phalcon:8.4-phalcon5.9

# For PHP 8.2 + Phalcon 5.7.0 (Alpine-based)
docker build -t fvildoso/php-phalcon:8.2.19-phalcon5.7.0-alpine ./php8.2-phalcon5

# For PHP 7.3 + Phalcon 3.4.5 (Alpine-based)
docker build -t fvildoso/php-phalcon:7.3.33-phalcon3.4.5-alpine ./php7.3-phalcon3
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.