ARG PHP_VERSION=8.4.8

FROM php:${PHP_VERSION}-fpm-alpine3.22 AS base

ARG PHP_VERSION=8.4.8
ARG PHALCON_VERSION=v5.9.3
ARG TARGETARCH
ARG TARGETPLATFORM

# Copiar el instalador de extensiones PHP (la herramienta más confiable)
COPY --from=ghcr.io/mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Instalar dependencias del sistema
RUN apk update && apk upgrade && \
    apk add --no-cache \
        bash \
        git \
        curl

# Instalar extensiones PHP usando el instalador oficial
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
        psr \
        zip

# Configurar PHP-FPM básico
RUN mkdir -p /var/log/php-fpm && \
    chown www-data:www-data /var/log/php-fpm

FROM base AS dev

# Copiar Composer
COPY --from=composer/composer:2 /usr/bin/composer /usr/bin/composer

# Instalar dependencias de desarrollo
RUN apk add --no-cache \
        autoconf \
        g++ \
        gcc \
        libc-dev \
        make \
        pkgconf \
        re2c

# Instalar zephir_parser y configurar memoria para compilación
RUN install-php-extensions zephir_parser && \
    echo 'memory_limit = -1' > /usr/local/etc/php/conf.d/90-memory.ini

WORKDIR /srv

FROM dev AS phalcon-builder

# Descargar código fuente de Phalcon
RUN git clone --depth=1 --branch=${PHALCON_VERSION} https://github.com/phalcon/cphalcon.git /srv

# Instalar Zephir y compilar Phalcon
RUN set -ex && \
    rm -rf /srv/vendor && \
    find /srv -type f -iname '*.dep' -exec rm {} \; && \
    \
    composer global require phalcon/zephir:dev-development && \
    \
    if [ "${TARGETARCH}" = "arm64" ] || [ "${TARGETARCH}" = "aarch64" ]; then \
        echo "🏗️ Compilando para ARM64 (memoria limitada)"; \
        /root/.composer/vendor/bin/zephir fullclean; \
        /root/.composer/vendor/bin/zephir build --no-dev; \
    else \
        echo "🏗️ Compilando para AMD64 (optimizado)"; \
        /root/.composer/vendor/bin/zephir fullclean; \
        /root/.composer/vendor/bin/zephir build; \
    fi && \
    \
    ls -la /srv/ext/modules/phalcon.so && \
    echo "✅ Phalcon compilado exitosamente para ${TARGETARCH}"

FROM base AS production

# Metadatos de la imagen
LABEL org.opencontainers.image.title="Phalcon ${PHALCON_VERSION} with PHP ${PHP_VERSION}" \
      org.opencontainers.image.description="Docker image with Phalcon and PHP on Alpine Linux" \
      org.opencontainers.image.version="${PHALCON_VERSION}-php${PHP_VERSION}"

# Copiar la extensión compilada
COPY --from=phalcon-builder /srv/ext/modules/phalcon.so /tmp/phalcon.so

# Instalar Phalcon y limpiar
RUN set -ex && \
    mv /tmp/phalcon.so "$(php-config --extension-dir)/phalcon.so" && \
    echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/70-phalcon.ini && \
    \
    php -m | grep phalcon && \
    echo "✅ Phalcon ${PHALCON_VERSION} instalado correctamente" && \
    \
    php -m | grep -E "(pdo|gd|imagick|memcached|phalcon|psr)" && \
    echo "🎉 Todas las extensiones verificadas" && \
    \
    rm -rf /tmp/* /var/tmp/* /var/cache/apk/*

# Healthcheck simple
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD php-fpm-healthcheck || exit 1

EXPOSE 9000

CMD ["php-fpm"]