FROM wordpress:php8.3-fpm
USER root
RUN apt-get update && apt-get install -y autoconf gcc make build-essential pkg-config && pecl install redis && docker-php-ext-enable redis
USER www-data
