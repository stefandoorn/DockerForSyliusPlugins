FROM php:7.2-fpm-alpine3.8

# System utilities
RUN set -xe \
    && apk add --no-cache \
        acl \
        git

# PHP extensions required by Sylius
RUN set -xe \
    && apk add --no-cache --virtual .build-deps \
        coreutils \
        freetype-dev \
        icu-dev \
        libjpeg-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
        --with-gd \
    && docker-php-ext-configure intl --enable-intl \
    && docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql \
    && docker-php-ext-install -j "$(nproc)" \
        exif \
        fileinfo \
        gd \
        intl \
        mysqli \
        opcache \
        pdo_mysql \
    && runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --virtual .run-deps $runDeps \
    && apk del .build-deps \
    && rm -rf /tmp/*

# Xdebug
RUN set -xe \
    && apk add --no-cache --virtual .phpize_deps \
        $PHPIZE_DEPS \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && apk del .phpize_deps \
    && rm -rf /tmp/*

# Yarn
RUN set -xe \
    && apk add --no-cache \
        python2 \
        yarn

# Switch app environment script
COPY switch-env /usr/local/bin/switch-env
RUN chmod +x /usr/local/bin/switch-env

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

USER www-data

RUN set -xe \
    && composer global require "hirak/prestissimo:^0.3" --prefer-dist --no-progress --no-suggest --classmap-authoritative \
    && composer clear-cache

WORKDIR /home/sylius
