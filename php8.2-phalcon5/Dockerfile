FROM php:8.2.19-fpm-alpine3.18

# Variables de entorno
ENV PHP_VERSION=8.2.19 \
    PHALCON_VERSION=5.7.0

# Verificar versión PHP
RUN php -v && \
    if [ $(php -r "echo PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION;") != "8.2" ]; then \
        echo "Error: PHP version is not 8.2" && exit 1; \
    fi

# Instalar dependencias necesarias
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        $PHPIZE_DEPS \
        libxml2-dev \
        libzip-dev \
        curl-dev \
        libwebp-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        oniguruma-dev \
        imagemagick \
        imagemagick-dev \
        pkgconf

# Instalar extensiones PHP
RUN docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        bcmath \
        curl

# Instalar Imagick
RUN apk add --no-cache imagemagick-dev && \
    pecl install imagick && \
    docker-php-ext-enable imagick

# Instalar Phalcon (requiere pecl/psr)
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS && \
    pecl install psr && \
    docker-php-ext-enable psr && \
    pecl install phalcon-${PHALCON_VERSION} && \
    docker-php-ext-enable phalcon && \
    apk del .build-deps

# Configurar GD
RUN docker-php-ext-configure gd \
        --enable-gd \
        --with-webp \
        --with-jpeg \
        --with-freetype \
        --enable-gd-jis-conv && \
    docker-php-ext-install -j$(nproc) gd

# Limpiar caché
RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*

CMD ["php-fpm"]