# from https://www.drupal.org/docs/8/system-requirements/drupal-8-php-requirements
FROM php:7.1-fpm

ARG WEB_USER
ARG WEB_GROUP
ARG PHP_ROOT_DIR

# install the PHP extensions we need
RUN set -ex; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libpng-dev \
		curl \
		wget \
		git-core \
		nano \
		vim \
    	libmemcached-dev \
    	libz-dev \
    	libpq-dev \
    	libjpeg-dev \
    	libpng-dev \
    	libfreetype6-dev \
    	libicu-dev \
    	libssl-dev \
    	libmcrypt-dev \
		unzip \
		mysql-client \
	; \
	\
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-install -j "$(nproc)" \
		gd \
		mysqli \
		mcrypt \
		intl \
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
	rm -rf /var/lib/apt/lists/*
# Install Memcached
RUN curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/php7.tar.gz" && \
mkdir -p memcached && \
tar -C memcached -zxvf /tmp/memcached.tar.gz --strip 1 && \
( \
    cd memcached && \
    phpize && \
    ./configure && \
    make -j$(nproc) && \
    make install \
) && \
rm -r memcached && \
rm /tmp/memcached.tar.gz && \
docker-php-ext-enable memcached

# set recommended PHP.ini settings
RUN { \
        echo 'memory_limit=512M'; \
        echo 'display_errors=On'; \
        echo 'display_startup_errors=On'; \
        echo 'realpath_cache_size=On'; \
        echo 'max_input_vars=4000'; \
        echo 'max_execution_time=6000'; \
        echo 'max_input_time=6000'; \
				echo 'opcache.memory_consumption=192'; \
				echo 'opcache.interned_strings_buffer=8'; \
				echo 'opcache.max_accelerated_files=4096'; \
				echo 'opcache.revalidate_freq=60'; \
				echo 'opcache.fast_shutdown=1'; \
				echo 'opcache.enable_cli=1'; \
				echo 'mysql.connect_timeout=600'; \
				echo 'default_socket_timeout=600'; \
	} > /usr/local/etc/php/conf.d/php-custom.ini
# install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer



# install drush
#WORKDIR /usr/local/bin/
#RUN wget -O drush https://github.com/drush-ops/drush/releases/download/8.1.17/drush.phar && chmod +x drush
RUN composer global require hirak/prestissimo
RUN composer global require drush/drush:8.*
ENV PATH /root/.composer/vendor/bin:$PATH

# install mailhog
RUN curl -Lsf 'https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz' | tar -C '/usr/local' -xvzf -
ENV PATH /usr/local/go/bin:$PATH
RUN go get github.com/mailhog/mhsendmail
RUN cp /root/go/bin/mhsendmail /usr/bin/mhsendmail
RUN echo 'sendmail_path = /usr/bin/mhsendmail --smtp-addr mailhog:1025' > /usr/local/etc/php/conf.d/mailhog.ini

COPY ./conf/www.conf ${PHP_ROOT_DIR}/php-fpm.d/www.conf

RUN usermod -u 1000 ${WEB_USER} \
 && groupmod -g 1000 ${WEB_GROUP} \
 && chgrp -R staff ${PHP_ROOT_DIR}/php-fpm.d/www.conf
