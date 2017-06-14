FROM php:7.1.6-apache

# Install git, unzip, filebeat, postfix
RUN apt-get update -qq \
 && apt-get install -y --no-install-recommends \
    git \
    sendmail \
    unzip \
 && apt-get clean autoclean \
 && apt-get autoremove --yes \
 && rm -rf /var/lib/{apt,dpkg,cache,log}/

# Main PHP ini
COPY php.ini /usr/local/etc/php/

#
# zlib1g-dev libicu-dev g++ for intl
# libmemcached-dev for memcached
# libgd-dev libmagickwand-dev for gd (images)
# xdebug for phpunit coverage
#  
RUN apt-get update -qq \
 && apt-get install -y --no-install-recommends \
    g++ \
    libgd-dev \
    libicu-dev \
    libmagickwand-dev \
    libmemcached-dev \
    libz-dev \
    zlib1g-dev \
 && a2enmod rewrite \
 && export \
    CFLAGS="$PHP_CFLAGS" \
    CPPFLAGS="$PHP_CPPFLAGS" \
    LDFLAGS="$PHP_LDFLAGS" \
 && pecl install \
    imagick \
    memcached \
    xdebug \
 && docker-php-ext-configure \
    gd \
    --with-freetype-dir=/usr/include/ \
    --with-gd \
    --with-jpeg-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
 && docker-php-ext-configure \
    intl \
 && docker-php-ext-enable \
    imagick \
    memcached \
 && docker-php-ext-install \
    gd \
    intl \
    pdo_mysql \
    zip \
 && apt-get clean autoclean \
 && apt-get autoremove --yes \
 && rm -rf /var/lib/{apt,dpkg,cache,log}/

# Install composer 
ENV PATH=/composer/vendor/bin:$PATH \
    COMPOSER_HOME=/composer \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_VERSION=1.4.2

RUN curl -s -f -L -o /tmp/installer.php https://raw.githubusercontent.com/composer/getcomposer.org/da290238de6d63faace0343efbdd5aa9354332c5/web/installer \
 && php -r " \
    \$signature = '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410'; \
    \$hash = hash('SHA384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
        unlink('/tmp/installer.php'); \
        echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
        exit(1); \
    }" \
 && php /tmp/installer.php \
    --no-ansi \
    --install-dir=/usr/bin \
    --filename=composer \
    --version=${COMPOSER_VERSION} \
 && rm /tmp/installer.php

# Override base image CMD command
COPY docker-entrypoint.sh /usr/local/bin/

RUN chmod -R 755 /usr/local/bin/docker-entrypoint.sh
 
ENTRYPOINT [ "docker-entrypoint.sh" ]

CMD [ "apache2-foreground" ]
