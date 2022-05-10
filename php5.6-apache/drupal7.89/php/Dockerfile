#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

# from https://www.drupal.org/docs/system-requirements/php-requirements
FROM php:5.6-apache

# install the PHP extensions we need
RUN set -eux; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libwebp-dev \
		libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg=/usr \
		--with-webp \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 7.89
ENV DRUPAL_MD5 924e707e3b1e8269fe623cf2f22ee710

RUN cd /var/www/html \
	set -eux; \
	curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz; \
	echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c -; \
	tar -xz --strip-components=1 -f drupal.tar.gz; \
	rm drupal.tar.gz; \
	chown -R www-data:www-data sites modules themes

# vim:set ft=dockerfile:

RUN \ 
	apt-get update && \
	apt-get install libldap2-dev -y && \
	rm -rf /var/lib/apt/lists/* && \
	docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
	docker-php-ext-install ldap

# RUN pecl install -f xdebug \
#	&& docker-php-ext-enable xdebug

#RUN echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.client_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.mode = debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.start_with_request = yes" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.client_port = 9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
#	echo "xdebug.idekey = VSCODE" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

RUN pecl install -f apcu-4.0.11 \
    && docker-php-ext-enable apcu

RUN echo "extension=apcu.so" > /usr/local/etc/php/php.ini
RUN echo "apc.enable_cli=1" > /usr/local/etc/php/php.ini
RUN echo "apc.enable=1" > /usr/local/etc/php/php.ini

RUN \ 
	apt-get update && \
	apt-get install git -y

RUN git clone https://github.com/xdebug/xdebug.git \
	&& cd xdebug \
	&& git checkout tags/XDEBUG_2_5_5 \
	&& phpize \
	&& ./configure --enable-xdebug \
	&& make \
	&& make install

RUN docker-php-ext-enable xdebug

RUN echo "xdebug.remote_enable = 1" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.client_host = host.docker.internal" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.mode = debug" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.start_with_request = yes" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.client_port = 9000" >> /usr/local/etc/php/php.ini && \
	echo "xdebug.idekey = VSCODE" >> /usr/local/etc/php/php.ini