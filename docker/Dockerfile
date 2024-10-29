ARG PHP_MINOR_VERSION=8.2
# Use Debian Bookworm as the base image
FROM php:${PHP_MINOR_VERSION}-fpm-bookworm AS builder
ARG PHP_MINOR_VERSION=8.2
ARG PATCH_VERSION=25
ARG PHP_VERSION=${PHP_MINOR_VERSION}.${PATCH_VERSION}

# Set build argument for PHP version with a default value
# Set environment variable to avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary build dependencies and tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libxml2-dev \
    libssl-dev \
    libcurl4-openssl-dev \
    libjpeg-dev \
    libpng-dev \
    libonig-dev \
    libzip-dev \
    libicu-dev \
    wget \
    tar \
    autoconf \
    bison \
    re2c \
    pkg-config \
    libsqlite3-dev \
    libreadline-dev \
    libargon2-dev \
    libbz2-dev \
    libfreetype6-dev \
    libgmp-dev \
    libldap2-dev \
    libmemcached-dev \
    libxslt1-dev \
    libpq-dev \
    zlib1g-dev \
    ca-certificates \
    zip unzip \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /usr/src

# Download and extract PHP source code
RUN wget https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz \
    && tar -xzf php-${PHP_VERSION}.tar.gz \
    && rm php-${PHP_VERSION}.tar.gz

# Set working directory to PHP source
WORKDIR /usr/src/php-${PHP_VERSION}

# Configure PHP with desired options, enabling FPM and debug symbols
RUN ./configure \
        --enable-fpm \
        --enable-debug \
        --with-openssl \
        --with-curl \
        --with-zlib \
        --with-zip \
        --with-iconv \
        --with-mysqli \
        --with-pdo-mysql \
        --with-pdo-pgsql \
        --with-sqlite3 \
        --with-bz2 \
        --with-readline \
        --enable-mbstring \
        --enable-zip \
        --enable-soap \
        --enable-intl \
        --with-gd \
        --enable-exif \
        --with-freetype \
        --with-jpeg \
        --with-png \
        --with-xsl \
        --with-gettext \
        --with-gmp \
        CFLAGS="-g -O2 -DDEBUG_ZEND=2" \
    && make -j"$(nproc)" \
    && make install;

# Create directories for PHP-FPM
RUN mkdir -p /var/run/php /var/log/php-fpm

FROM php:${PHP_MINOR_VERSION}-fpm-bookworm

# Install necessary libraries for PHP extensions
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev \
    default-mysql-client \
    libxslt1.1 \
    zip unzip libzip-dev \
    # The GNU debugger for segfaults
    gdb \
    && rm -rf /var/lib/apt/lists/*

# Copy PHP binaries and libraries from the builder
COPY --from=builder /usr/local/bin/php /usr/local/bin/php
COPY --from=builder /usr/local/etc/php /usr/local/etc/php
COPY --from=builder /usr/local/lib/php /usr/local/lib/php
COPY --from=builder /usr/local/include/php /usr/local/include/php

# Copy the default PHP-FPM configuration
RUN cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
# Check debug build
RUN php -i | grep 'Debug Build'
# Check for CFLAGS of debugging build
RUN php -i | grep 'CFLAGS'

# Expose PHP-FPM port
EXPOSE 9000

# Set entrypoint to PHP-FPM
CMD ["php-fpm"]


