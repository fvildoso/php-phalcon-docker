FROM php:8.4.8-fpm-alpine3.22

# Variables de entorno
ENV PHP_VERSION=8.4.8
ENV PHALCON_VERSION=5.9.3

# Verificar versión PHP
RUN php -v && \
    if [ $(php -r "echo PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION;") != "8.4" ]; then \
        echo "Error: PHP version is not 8.4" && exit 1; \
    fi

# Definir dependencias de desarrollo
ENV PHPIZE_DEPS \
    autoconf \
    dpkg-dev dpkg \
    file \
    g++ \
    gcc \
    libc-dev \
    make \
    pkgconf \
    re2c

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
        gettext-dev \
        openssl-dev \
        libmemcached-dev \
        zlib-dev

# Instalar extensiones PHP básicas
RUN docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        bcmath \
        curl \
        soap \
        fileinfo \
        gettext \
        mbstring \
        zip

# Configurar y instalar GD
RUN docker-php-ext-configure gd \
        --enable-gd \
        --with-webp \
        --with-jpeg \
        --with-freetype \
        --enable-gd-jis-conv && \
    docker-php-ext-install -j$(nproc) gd

# Instalar extensiones vía PECL
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS && \
    pecl install imagick && \
    docker-php-ext-enable imagick && \
    pecl install memcached && \
    docker-php-ext-enable memcached && \
    pecl install phalcon-${PHALCON_VERSION} && \
    docker-php-ext-enable phalcon && \
    apk del .build-deps

# Verificar extensiones instaladas
RUN php -m | grep -E "(curl|fileinfo|gettext|gd|imagick|pdo|openssl|mbstring|soap|memcached|phalcon|bcmath)" || \
    (echo "Error: Faltan extensiones requeridas" && exit 1)

# Limpiar caché
RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/* && \
    rm -rf /var/tmp/*

CMD ["php-fpm"]