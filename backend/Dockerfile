#FROM php:8.1-fpm-alpine
#RUN set -eux \
#    && apk add --no-cache \
#        c-client \
#        ca-certificates \
#        freetds \
#        freetype \
#        gettext \
#        gmp \
#        icu-libs \
#        imagemagick \
#        imap \
#        libffi \
#        libgmpxx \
#        libintl \
#        libjpeg-turbo \
#        libpng \
#        libpq \
#        librdkafka \
#        libssh2 \
#        libstdc++ \
#        libtool \
#        libxpm \
#        libxslt \
#        libzip \
#        lz4-libs \
#        make \
#        rabbitmq-c \
#        tidyhtml \
#        tzdata \
#        unixodbc \
#        vips \
#        yaml \
#        zstd-libs \
#    && true
## Install composer and add its bin to the PATH.
#RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer
#RUN composer self-update --2
#WORKDIR /var/www/html
#COPY . /var/www/html
#
#RUN chown -R www-data:www-data /var/www/html
#CMD ["/bin/sh", "-c", "composer install"]
FROM php:8-fpm-alpine
ARG UID
ARG GID
ENV UID=${UID}
ENV GID=${GID}

RUN mkdir -p /usr/share/nginx/html

WORKDIR /usr/share/nginx/html

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# MacOS staff group's gid is 20, so is the dialout group in alpine linux. We're not using it, let's just remove it.
RUN delgroup dialout

RUN addgroup -g ${GID} --system laravel
RUN adduser -G laravel --system -D -s /bin/sh -u ${UID} laravel

RUN sed -i "s/user = www-data/user = laravel/g" /usr/local/etc/php-fpm.d/www.conf
RUN sed -i "s/group = www-data/group = laravel/g" /usr/local/etc/php-fpm.d/www.conf
RUN echo "php_admin_flag[log_errors] = on" >> /usr/local/etc/php-fpm.d/www.conf

RUN docker-php-ext-install pdo pdo_mysql

RUN mkdir -p /usr/src/php/ext/redis \
    && curl -L https://github.com/phpredis/phpredis/archive/5.3.4.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
    && echo 'redis' >> /usr/src/php-available-exts \
    && docker-php-ext-install redis

USER laravel

CMD ["php-fpm", "-y", "/usr/local/etc/php-fpm.conf", "-R"]
