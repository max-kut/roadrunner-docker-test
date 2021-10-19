FROM php:7.4-cli-alpine
RUN apk update \
    && apk add --no-cache  \
        oniguruma \
        oniguruma-dev \
        libgcrypt \
        libgcrypt-dev \
        ca-certificates \
        pcre-dev \
        curl \
        libcurl \
        openssl-dev \
        wget \
        libmcrypt-dev \
        freetype-dev \
        libpng-dev \
        libjpeg-turbo-dev \
        libxml2-dev \
        autoconf \
        g++ \
        imagemagick-dev \
        imagemagick \
        libtool \
        make \
        icu-dev \
        libxslt-dev \
        zip \
        nano \
        patch \
        gnu-libiconv \
        bash \
        libzip-dev \
        htop \
        ${PHPIZE_DEPS} \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install \
        mbstring \
        opcache \
        iconv \
        pdo \
        gd \
        dom \
        exif \
        intl \
        tokenizer \
        zip \
        pcntl\
        sockets

RUN docker-php-source delete \
    && apk del ${BUILD_DEPENDS}

RUN docker-php-ext-install -j$(nproc) pdo_mysql \
    && docker-php-ext-enable pdo_mysql

ARG WITH_XDEBUG
RUN (test ! -z "${WITH_XDEBUG}" \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
       && pecl install xdebug \
       && docker-php-ext-enable xdebug \
       && apk del -f .build-deps \
    ) || true

RUN EXPECTED_COMPOSER_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig) \
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === '${EXPECTED_COMPOSER_SIGNATURE}') { echo 'Composer.phar Installer verified'; } else { echo 'Composer.phar Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && docker-php-source delete \
    && composer clear-cache

ARG PHP_INI_MODE="production"

RUN cp "${PHP_INI_DIR}/php.ini-${PHP_INI_MODE}" "${PHP_INI_DIR}/php.ini"

RUN echo 'opcache.enable_cli=on' | tee "${PHP_INI_DIR}/conf.d/90-user.ini"

WORKDIR /var/www/app

CMD ["sleep","3600"]
