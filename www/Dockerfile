FROM php:7.2-apache

RUN docker-php-ext-install mysqli
RUN docker-php-ext-enable mysqli

RUN pecl install xdebug-2.8.1
RUN docker-php-ext-enable xdebug
