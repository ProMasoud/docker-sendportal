FROM php:8.1-fpm-alpine

RUN apk --update add wget \
    curl \
    git \
    grep \
    build-base \
    libmemcached-dev \
    libmcrypt-dev \
    libxml2-dev \
    imagemagick-dev \
    pcre-dev \
    libtool \
    make \
    autoconf \
    g++ \
    cyrus-sasl-dev \
    libgsasl-dev \
    supervisor

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions memcached mcrypt imagick redis mysqli opcache pcntl pdo_mysql sodium zip curl;

RUN mkdir -p /var/www

COPY opcache.ini $PHP_INI_DIR

# Setup sendportal
WORKDIR /app
RUN git clone https://github.com/ProMasoud/sendportal.git .
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer install --ignore-platform-reqs
RUN chmod +x /app/artisan
RUN chown -R www-data:www-data /app/storage

# Publish laravel horizon assets
RUN /app/artisan horizon:publish

# Add SendPortal cron jobs that runs every minute
RUN echo '* * * * * php /app/artisan schedule:run >> /dev/null 2>&1' > /etc/crontabs/root
RUN echo '* * * * * php /app/artisan sp:campaigns:dispatch >> /dev/null 2>&1' >> /etc/crontabs/root

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
