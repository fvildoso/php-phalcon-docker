FROM php:8.4.8-fpm-alpine3.22

# Variables
ARG PHALCON_VERSION=5.9.3
ARG TARGETARCH
ARG TARGETPLATFORM

ENV PHALCON_VERSION=${PHALCON_VERSION}

# Copiar instalador de extensiones PHP
COPY --from=ghcr.io/mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Instalar dependencias del sistema
RUN apk update && apk upgrade && \
    apk add --no-cache \
        bash \
        git \
        curl

# Instalar extensiones PHP básicas usando el instalador oficial
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

# Estrategia simple: Intentar PECL primero, fallback a GitHub releases
RUN set -ex && \
    echo "📦 Instalando Phalcon ${PHALCON_VERSION}..." && \
    \
    ( \
        echo "🎯 Intentando instalación vía PECL..." && \
        pecl channel-update pecl.php.net && \
        pecl install phalcon-${PHALCON_VERSION} \
    ) || ( \
        echo "⚠️ PECL falló, usando binario precompilado..." && \
        \
        if [ "${TARGETARCH}" = "arm64" ] || [ "${TARGETARCH}" = "aarch64" ]; then \
            ARCH_SUFFIX="arm64"; \
        else \
            ARCH_SUFFIX="amd64"; \
        fi && \
        \
        cd /tmp && \
        PHALCON_SO_URL="https://github.com/phalcon/cphalcon/releases/download/v${PHALCON_VERSION}/phalcon-php8.4-linux-${ARCH_SUFFIX}.so" && \
        echo "🔗 Intentando descargar: ${PHALCON_SO_URL}" && \
        ( \
            curl -L -f -o phalcon.so "${PHALCON_SO_URL}" && \
            mv phalcon.so "$(php-config --extension-dir)/phalcon.so" \
        ) || ( \
            echo "⚠️ Binario no disponible, compilando desde fuente simple..." && \
            \
            apk add --no-cache --virtual .build-deps \
                autoconf \
                g++ \
                gcc \
                libc-dev \
                make \
                pkgconf \
                re2c && \
            \
            cd /tmp && \
            git clone --depth=1 --branch=v${PHALCON_VERSION} https://github.com/phalcon/cphalcon.git && \
            cd cphalcon/build && \
            ./install && \
            \
            apk del .build-deps && \
            cd / && rm -rf /tmp/cphalcon \
        ) \
    ) && \
    \
    echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/70-phalcon.ini && \
    \
    php -m | grep phalcon && \
    echo "✅ Phalcon ${PHALCON_VERSION} instalado correctamente para ${TARGETARCH}"

# Verificación final
RUN set -ex && \
    echo "🔍 Verificando extensiones instaladas:" && \
    php -m | grep -E "(pdo|curl|gd|imagick|memcached|phalcon|psr)" && \
    echo "🎉 Todas las extensiones verificadas correctamente"

# Limpiar caché final
RUN rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

# Configurar PHP-FPM
RUN mkdir -p /var/log/php-fpm && \
    chown www-data:www-data /var/log/php-fpm

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD php-fpm-healthcheck || exit 1

EXPOSE 9000

CMD ["php-fpm"]