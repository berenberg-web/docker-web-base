FROM php:5.6.30-apache

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
# libaio-dev for oci
# zlib1g-dev libicu-dev g++ for intl
# libpq-dev for pdo_pgsql, pgsql
# libmemcached-dev for memcached
# libgd-dev libmagickwand-dev for gd (images)
# xdebug for phpunit coverage
#  
RUN apt-get install -y --no-install-recommends \
    g++ \
    libaio-dev \
    libgd-dev \
    libicu-dev \
    libmagickwand-dev \
    libmemcached-dev \
    libpq-dev \
    libz-dev \
    zlib1g-dev \
 && a2enmod rewrite \
 && export \
    CFLAGS="$PHP_CFLAGS" \
    CPPFLAGS="$PHP_CPPFLAGS" \
    LDFLAGS="$PHP_LDFLAGS" \
 && pecl install \
    imagick \
    memcached-2.2.0 \
    xdebug \
 && docker-php-ext-configure \
    gd \
    --with-freetype-dir=/usr/include/ \
    --with-gd \
    --with-jpeg-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
 && docker-php-ext-configure \
    intl \
 && docker-php-ext-configure \
    pgsql \
    -with-pgsql=/usr/local/pgsql \
 && docker-php-ext-enable \
    imagick \
    memcached \
 && docker-php-ext-install \
    gd \
    intl \
    pdo_mysql \
    pdo_pgsql \
    pgsql \
    zip \
 && apt-get clean autoclean \
 && apt-get autoremove --yes \
 && rm -rf /var/lib/{apt,dpkg,cache,log}/

# Install composer 
ENV PATH=/composer/vendor/bin:$PATH \
    COMPOSER_HOME=/composer \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_VERSION=1.4.2

COPY composer_installer.php /tmp/installer.php

RUN php /tmp/installer.php \
    --no-ansi \
    --install-dir=/usr/bin \
    --filename=composer \
    --version=${COMPOSER_VERSION} \
 && rm /tmp/installer.php

# oci
COPY oracle/* /tmp/
    
RUN unzip /tmp/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
 && unzip /tmp/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
 && rm -f /tmp/instantclient-* \
 && ln -s /usr/local/instantclient_12_1 /usr/local/instantclient \
 && ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so \
 && echo 'instantclient,/usr/local/instantclient' | pecl install oci8-2.0.12

# Override base image CMD command
COPY docker-entrypoint.sh /usr/local/bin/

RUN chmod -R 755 /usr/local/bin/docker-entrypoint.sh
 
ENTRYPOINT [ "docker-entrypoint.sh" ]

CMD [ "apache2-foreground" ]
