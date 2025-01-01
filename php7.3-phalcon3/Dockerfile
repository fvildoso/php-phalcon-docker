FROM php:7.3.33-fpm-alpine3.14

# Variables de entorno
ENV PHP_VERSION=7.3.33 \
    PHALCON_VERSION=3.4.5

# Instalar dependencias necesarias
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        $PHPIZE_DEPS \
        libxml2-dev \
        libzip-dev \
        curl-dev \
        git \
        autoconf \
        gcc \
        g++ \
        make \
        re2c

# Instalar extensiones PHP comunes
RUN docker-php-ext-install -j$(nproc) \
        pdo \
        pdo_mysql \
        bcmath \
        json \
        mbstring

# Instalar Phalcon
RUN git clone https://github.com/phalcon/cphalcon.git -b v${PHALCON_VERSION} /tmp/cphalcon && \
    cd /tmp/cphalcon/build && \
    ./install && \
    docker-php-ext-enable phalcon

# Limpiar caché
RUN rm -rf /var/cache/apk/* && \
    rm -rf /tmp/*

CMD ["php-fpm"]