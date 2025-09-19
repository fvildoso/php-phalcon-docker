ARG PHP_VERSION=8.4
FROM php:${PHP_VERSION}-fpm AS base

# Variables
ARG PHALCON_VERSION=5.9.4
ARG TARGETARCH
ARG TARGETPLATFORM
ENV PHALCON_VERSION=${PHALCON_VERSION}

# Copiar instalador de extensiones PHP
COPY --from=ghcr.io/mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Instalar dependencias del sistema (incluyendo herramientas de compilación)
# Multi-arch compatible package installation
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        bash \
        git \
        curl \
        autoconf \
        g++ \
        gcc \
        libc6-dev \
        make \
        pkg-config \
        re2c \
        libzip-dev \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libmagickwand-dev \
        libmemcached-dev \
        zlib1g-dev \
        libicu-dev && \
    echo "🏗️  Building for: ${TARGETPLATFORM} (${TARGETARCH})" && \
    uname -m

# Instalar extensiones PHP básicas
RUN install-php-extensions \
        pdo_mysql \
        bcmath \
        curl \
        soap \
        fileinfo \
        gettext \
        gd \
        imagick \
        mbstring \
        memcached \
        opcache \
        zip

# Instalar PSR (requerido para Phalcon)
RUN install-php-extensions psr

# Configurar flags de compilación para PHP 8.4 compatibility
ENV CFLAGS="-Wno-incompatible-pointer-types -Wno-int-conversion"
ENV CXXFLAGS="-Wno-incompatible-pointer-types -Wno-int-conversion"

# Instalar Phalcon vía PECL
RUN set -ex && \
    echo "📦 Instalando Phalcon ${PHALCON_VERSION} vía PECL..." && \
    pecl channel-update pecl.php.net && \
    (pecl install phalcon-${PHALCON_VERSION} || \
     pecl install phalcon || \
     (echo "Instalando desde código fuente..." && \
      git clone --depth=1 --branch=master https://github.com/phalcon/cphalcon.git /tmp/cphalcon && \
      cd /tmp/cphalcon && \
      git submodule update --init --recursive && \
      cd /tmp/cphalcon/build && \
      ./install --phpize /usr/local/bin/phpize --php-config /usr/local/bin/php-config && \
      rm -rf /tmp/cphalcon)) && \
    EXT_DIR=$(php-config --extension-dir) && \
    echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/70-phalcon.ini && \
    ls -la "${EXT_DIR}" && \
    php -m | grep phalcon && \
    echo "✅ Phalcon ${PHALCON_VERSION} instalado correctamente" || \
    ( \
        echo "❌ Error: Falló la instalación de Phalcon" && \
        exit 1 \
    )

# Limpiar dependencias de compilación (opcional, para reducir el tamaño de la imagen)
RUN apt-get purge -y \
        autoconf \
        g++ \
        gcc \
        libc6-dev \
        make \
        pkg-config \
        re2c && \
    apt-get autoremove -y && \
    apt-get autoclean

# Verificación final
RUN set -ex && \
    echo "🔍 Verificando extensiones instaladas:" && \
    php -m | grep -E "(pdo|curl|gd|imagick|memcached|phalcon|psr)" && \
    echo "🎉 Todas las extensiones verificadas correctamente"

# Limpiar caché final
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/apt/archives/*

# Configurar PHP-FPM
RUN mkdir -p /var/log/php-fpm && \
    chown www-data:www-data /var/log/php-fpm

# Instalar php-fpm-healthcheck
RUN curl -o /usr/local/bin/php-fpm-healthcheck \
    https://raw.githubusercontent.com/renatomefi/php-fpm-healthcheck/master/php-fpm-healthcheck \
    && chmod +x /usr/local/bin/php-fpm-healthcheck

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD php-fpm-healthcheck || exit 1

EXPOSE 9000

CMD ["php-fpm"]