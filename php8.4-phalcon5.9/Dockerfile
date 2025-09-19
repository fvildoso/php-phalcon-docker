FROM php:8.4.8-fpm-alpine3.22

# Variables de entorno
ENV PHP_VERSION=8.4.8
ENV PHALCON_VERSION=5.9.3
ARG TARGETARCH
ARG TARGETPLATFORM

# Mostrar información de la plataforma de build
RUN echo "🏗️  Building for: ${TARGETPLATFORM:-unknown}" && \
    echo "📦 Architecture: ${TARGETARCH:-unknown}" && \
    uname -m

# Verificar versión PHP
RUN php -v && \
    if [ $(php -r "echo PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION;") != "8.4" ]; then \
        echo "❌ Error: PHP version is not 8.4" && exit 1; \
    else \
        echo "✅ PHP 8.4 verificado correctamente"; \
    fi

# Definir dependencias de desarrollo
ENV PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make pkgconf re2c"

# Instalar dependencias del sistema con optimizaciones por arquitectura
RUN apk update && apk upgrade && \
    apk add --no-cache \
        $PHPIZE_DEPS \
        git \
        libxml2-dev \
        libzip-dev \
        curl-dev \
        libwebp-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        oniguruma-dev \
        gettext-dev \
        openssl-dev \
        zlib-dev \
        imagemagick \
        imagemagick-dev \
        libmemcached-dev \
        cyrus-sasl-dev \
    && echo "✅ Dependencias del sistema instaladas"

# Instalar extensiones PHP básicas con optimizaciones por arquitectura
RUN set -ex && \
    if [ "${TARGETARCH}" = "arm64" ] || [ "${TARGETARCH}" = "aarch64" ]; then \
        PARALLEL_JOBS=$(($(nproc) > 2 ? 2 : $(nproc))); \
    else \
        PARALLEL_JOBS=$(nproc); \
    fi && \
    echo "🚀 Using ${PARALLEL_JOBS} parallel processes for ${TARGETARCH}" && \
    docker-php-ext-install -j${PARALLEL_JOBS} \
        pdo \
        pdo_mysql \
        bcmath \
        curl \
        soap \
        fileinfo \
        gettext \
        mbstring \
        zip && \
    docker-php-ext-configure gd \
        --enable-gd \
        --with-webp \
        --with-jpeg \
        --with-freetype && \
    docker-php-ext-install -j${PARALLEL_JOBS} gd && \
    echo "✅ Extensiones PHP básicas y GD instaladas"

# Instalar extensiones PECL con manejo de errores mejorado
RUN set -ex && \
    echo "📦 Instalando extensión PSR..." && \
    pecl install psr && \
    docker-php-ext-enable psr && \
    php -m | grep psr && \
    echo "✅ PSR instalado correctamente"

RUN set -ex && \
    echo "📦 Instalando ImageMagick..." && \
    pecl install imagick && \
    docker-php-ext-enable imagick && \
    php -m | grep imagick && \
    echo "✅ ImageMagick instalado correctamente"

RUN set -ex && \
    echo "📦 Instalando Memcached..." && \
    pecl install memcached && \
    docker-php-ext-enable memcached && \
    php -m | grep memcached && \
    echo "✅ Memcached instalado correctamente"

# Instalar Phalcon con manejo específico por arquitectura
RUN set -ex && \
    echo "📦 Instalando Phalcon ${PHALCON_VERSION} para ${TARGETARCH}..." && \
    case ${TARGETARCH} in \
        amd64) \
            echo "🏗️  Instalación optimizada para AMD64" && \
            pecl install phalcon-${PHALCON_VERSION} \
            ;; \
        arm64|aarch64) \
            echo "🏗️  Instalación para ARM64 (puede tardar más)" && \
            pecl install phalcon-${PHALCON_VERSION} \
            ;; \
        *) \
            echo "🏗️  Instalación genérica para ${TARGETARCH}" && \
            pecl install phalcon-${PHALCON_VERSION} \
            ;; \
    esac && \
    docker-php-ext-enable phalcon && \
    php -m | grep phalcon && \
    echo "✅ Phalcon ${PHALCON_VERSION} instalado correctamente para ${TARGETARCH}"

# Verificación final de todas las extensiones
RUN set -ex && \
    echo "🔍 Verificando todas las extensiones..." && \
    php -m | grep -i pdo && echo "✅ PDO" && \
    php -m | grep -i curl && echo "✅ cURL" && \
    php -m | grep -i gd && echo "✅ GD" && \
    php -m | grep -i imagick && echo "✅ ImageMagick" && \
    php -m | grep -i memcached && echo "✅ Memcached" && \
    php -m | grep -i phalcon && echo "✅ Phalcon" && \
    php -m | grep -i psr && echo "✅ PSR" && \
    echo "🎉 Todas las extensiones instaladas correctamente en ${TARGETPLATFORM}"

# Limpiar dependencias de build y caché
RUN set -ex && \
    echo "🧹 Limpiando caché y dependencias..." && \
    apk del $PHPIZE_DEPS && \
    rm -rf /var/cache/apk/* /tmp/* /var/tmp/* /usr/share/man ~/.pearrc && \
    echo "✅ Limpieza completada"

# Configurar PHP-FPM
RUN mkdir -p /var/log/php-fpm && \
    chown www-data:www-data /var/log/php-fpm && \
    echo "✅ PHP-FPM configurado"

# Información final del build
RUN set -ex && \
    echo "📋 Información del contenedor:" && \
    echo "🐘 PHP: $(php -v | head -n 1)" && \
    echo "🏗️  Plataforma: ${TARGETPLATFORM:-unknown}" && \
    echo "📦 Arquitectura: ${TARGETARCH:-unknown}" && \
    echo "🛠️  Sistema: $(uname -m)" && \
    echo "🎯 Phalcon: $(php -m | grep -i phalcon || echo 'No detectado')"

EXPOSE 9000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD php-fpm-healthcheck || exit 1

CMD ["php-fpm"]