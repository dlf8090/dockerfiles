FROM php:7.2-cli-alpine3.12

LABEL maintainer="donglifan<donggefor2018@163.com>"

# repositorie mirror、timezone
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/' /etc/apk/repositories ; \
    apk add --no-cache --virtual .timezone-deps tzdata; \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ; \
    echo 'Asia/Shanghai' > /etc/timezone; \
    rm -rf /var/cache/apk/* /tmp/* /var/tmp/* /var/lib/apt/lists/*

ARG SWOOLE_VERSION=4.5.2
ARG UNAME=dongge
ARG UID=1000
ARG GID=1000

RUN set -ex ; \
	apk add --no-cache --virtual .build-deps \
	    $PHPIZE_DEPS \
	    autoconf \
	    libaio-dev \
	    openssl-dev \
	    imagemagick-dev \
	    freetype-dev \
	    icu-dev \
	    curl-dev \
	    git \
	    libxml2-dev \
	    libmcrypt-dev \
	    libmemcached-dev \
		libjpeg-turbo-dev \
		libpng-dev \
        rabbitmq-c \
        rabbitmq-c-dev; \
	docker-php-ext-configure gd --with-png-dir=/usr/include --with-jpeg-dir=/usr/include --with-freetype-dir=/usr/include/freetype2; \
	docker-php-ext-configure intl; \
	docker-php-ext-install gd mysqli opcache pdo_mysql zip intl exif pcntl bcmath soap sockets; \
	pecl install redis-4.2.0; \
	pecl install memcached-3.1.3; \
	pecl install imagick-3.4.3; \
	pecl install mongodb-1.5.3; \
	pecl install mcrypt-1.0.1; \
    pecl install amqp-1.10.2; \
	# swoole
    curl -SL "https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz" -o swoole.tar.gz; \
    mkdir -p swoole; \
    tar -xf swoole.tar.gz -C swoole --strip-components=1; \
    rm swoole.tar.gz; \
    ( \
        cd swoole; \
        phpize; \
        ./configure --enable-mysqlnd --enable-openssl --enable-http2; \
        make -j$(nproc) ; \
        make install \
    ); \
    echo "swoole.use_shortname='Off'" >> /usr/local/etc/php/conf.d/50_swoole.ini; \
    rm -r swoole; \
    # solr
    git clone https://github.com/php/pecl-search_engine-solr.git --depth=1; \
    ( \
        cd pecl-search_engine-solr; \
        phpize; \
        ./configure; \
        make -j$(nproc) ; \
        make install \
    ); \
    rm -r pecl-search_engine-solr; \
	docker-php-ext-enable redis memcached imagick swoole solr mongodb mcrypt amqp;\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --virtual .epwk-phpexts-rundeps $runDeps bzip2; \
	apk del .build-deps; \
	rm -rf /var/cache/apk/* /tmp/* /var/tmp/* /var/lib/apt/lists/* /var/www/html

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Iconv fix: https://github.com/docker-library/php/issues/240#issuecomment-305038173
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ gnu-libiconv
ENV LD_PRELOAD=/usr/lib/preloadable_libiconv.so

COPY php7.2.ini /usr/local/etc/php/php.ini
COPY phpcs.phar /usr/local/bin/phpcs
COPY phpcbf.phar /usr/local/bin/phpcbf

# bash
#RUN apk add --no-cache \
#	bash

#RUN apk --no-cache add shadow; \
#    usermod -u 1000 www-data

RUN chmod +x /usr/local/bin/phpcs \
    && chmod +x /usr/local/bin/phpcbf \
    && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

# run as host user dongge
RUN adduser -D -u ${UID} ${UNAME}
USER ${UNAME}

WORKDIR /var/www
