# Dockerfile - alpine
# https://github.com/openresty/docker-openresty

FROM alpine:3.9

LABEL maintainer="Evan Wies <evan@neomantra.net>"

# Docker Build Arguments
ARG RESTY_VERSION="1.15.8.1rc1"
ARG RESTY_OPENSSL_VERSION="1.0.2r"
ARG RESTY_PCRE_VERSION="8.42"
ARG RESTY_J="1"
ARG RESTY_CONFIG_OPTIONS="\
    --with-file-aio \
    --with-http_addition_module \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_geoip_module=dynamic \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module=dynamic \
    --with-http_mp4_module \
    --with-http_random_index_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-http_xslt_module=dynamic \
    --with-ipv6 \
    --with-mail \
    --with-mail_ssl_module \
    --with-md5-asm \
    --with-pcre-jit \
    --with-sha1-asm \
    --with-stream \
    --with-stream_ssl_module \
    --with-threads \
    "
ARG RESTY_CONFIG_OPTIONS_MORE=""
ARG RESTY_ADD_PACKAGE_BUILDDEPS=""
ARG RESTY_ADD_PACKAGE_RUNDEPS=""
ARG RESTY_EVAL_PRE_CONFIGURE=""
ARG RESTY_EVAL_POST_MAKE=""

LABEL resty_version="${RESTY_VERSION}"
LABEL resty_openssl_version="${RESTY_OPENSSL_VERSION}"
LABEL resty_pcre_version="${RESTY_PCRE_VERSION}"
LABEL resty_config_options="${RESTY_CONFIG_OPTIONS}"
LABEL resty_config_options_more="${RESTY_CONFIG_OPTIONS_MORE}"
LABEL resty_add_package_builddeps="${RESTY_ADD_PACKAGE_BUILDDEPS}"
LABEL resty_add_package_rundeps="${RESTY_ADD_PACKAGE_RUNDEPS}"
LABEL resty_eval_pre_configure="${RESTY_EVAL_PRE_CONFIGURE}"
LABEL resty_eval_post_make="${RESTY_EVAL_POST_MAKE}"

# These are not intended to be user-specified
ARG _RESTY_CONFIG_DEPS="--with-openssl=/tmp/openssl-${RESTY_OPENSSL_VERSION} --with-pcre=/tmp/pcre-${RESTY_PCRE_VERSION}"

# ensure www-data user exists
RUN set -x \
	&& addgroup -g 82 -S www-data \
	&& adduser -u 82 -D -S -G www-data www-data
# 82 is the standard uid/gid for "www-data" in Alpine

# Start install supervisor
RUN apk add supervisor

# 1) Install apk dependencies
# 2) Download and untar OpenSSL, PCRE, and OpenResty
# 3) Build OpenResty
# 4) Cleanup

RUN apk add --no-cache --virtual .build-deps \
        build-base \
        curl \
        gd-dev \
        geoip-dev \
        libxslt-dev \
        linux-headers \
        make \
        perl-dev \
        readline-dev \
        zlib-dev \
        ${RESTY_ADD_PACKAGE_BUILDDEPS} \
    && apk add --no-cache \
        gd \
        geoip \
        libgcc \
        libxslt \
        zlib \
        ${RESTY_ADD_PACKAGE_RUNDEPS} \
    && cd /tmp \
    && if [ -n "${RESTY_EVAL_PRE_CONFIGURE}" ]; then eval $(echo ${RESTY_EVAL_PRE_CONFIGURE}); fi \
    && curl -fSL https://www.openssl.org/source/openssl-${RESTY_OPENSSL_VERSION}.tar.gz -o openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && tar xzf openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && curl -fSL https://ftp.pcre.org/pub/pcre/pcre-${RESTY_PCRE_VERSION}.tar.gz -o pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && tar xzf pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && curl -fSL https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz -o openresty-${RESTY_VERSION}.tar.gz \
    && tar xzf openresty-${RESTY_VERSION}.tar.gz \
    && cd /tmp/openresty-${RESTY_VERSION} \
# start install custom module
    && curl -fSL https://github.com/vozlt/nginx-module-vts/archive/v0.1.18.tar.gz -o nginx-module-vts-0.1.18.tar.gz \
    && tar xzf nginx-module-vts-0.1.18.tar.gz \
    && mv nginx-module-vts-0.1.18 nginx-module-vts \
	&& RESTY_CUSTOM_MODULE="--add-module=nginx-module-vts" \
# end install custom module
    && ./configure -j${RESTY_J} ${_RESTY_CONFIG_DEPS} ${RESTY_CONFIG_OPTIONS} ${RESTY_CUSTOM_MODULE} ${RESTY_CONFIG_OPTIONS_MORE} \
    && make -j${RESTY_J} \
    && make -j${RESTY_J} install \
    && cd /tmp \
    && if [ -n "${RESTY_EVAL_POST_MAKE}" ]; then eval $(echo ${RESTY_EVAL_POST_MAKE}); fi \
    && rm -rf \
        openssl-${RESTY_OPENSSL_VERSION} \
        openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
        openresty-${RESTY_VERSION}.tar.gz openresty-${RESTY_VERSION} \
        pcre-${RESTY_PCRE_VERSION}.tar.gz pcre-${RESTY_PCRE_VERSION} \
    && apk del .build-deps \
    && ln -sf /dev/stdout /usr/local/openresty/nginx/logs/access.log \
    && ln -sf /dev/stderr /usr/local/openresty/nginx/logs/error.log

# Start install php
ENV PHPIZE_DEPS \
		autoconf \
		dpkg-dev dpkg \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkgconf \
		re2c

# persistent / runtime deps
RUN apk add --no-cache \
		ca-certificates \
		curl \
		tar \
		xz \
# https://github.com/docker-library/php/issues/494
		openssl

ENV PHP_INI_DIR /usr/local/etc/php
RUN set -eux; \
	mkdir -p "$PHP_INI_DIR/conf.d"; \
# allow running as an arbitrary user (https://github.com/docker-library/php/issues/743)
	[ ! -d /var/www/html ]; \
	mkdir -p /var/www/html; \
	chown www-data:www-data /var/www/html; \
	chmod 777 /var/www/html

##<autogenerated>##
ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data --disable-cgi
##</autogenerated>##

# Apply stack smash protection to functions using local buffers and alloca()
# Make PHP's main executable position-independent (improves ASLR security mechanism, and has no performance impact on x86_64)
# Enable optimization (-O2)
# Enable linker optimization (this sorts the hash buckets to improve cache locality, and is non-default)
# Adds GNU HASH segments to generated executables (this is used if present, and is much faster than sysv hash; in this configuration, sysv hash is also generated)
# https://github.com/docker-library/php/issues/272
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ENV GPG_KEYS A917B1ECDA84AEC2B568FED6F50ABC807BD5DCD0 528995BFEDFBA7191D46839EF9BA0ADA31CBD89E 1729F83938DA44E27BA0F4D3DBDB397470D12172

ENV PHP_VERSION 7.1.29
ENV PHP_URL="https://www.php.net/get/php-7.1.29.tar.xz/from/this/mirror" PHP_ASC_URL="https://www.php.net/get/php-7.1.29.tar.xz.asc/from/this/mirror"
ENV PHP_SHA256="b9a9b094687edc2d9c9553d5531e38e249b569127cf3b32fe1c84280509746fb" PHP_MD5=""

RUN set -xe; \
	\
	apk add --no-cache --virtual .fetch-deps \
		gnupg \
	; \
	\
	mkdir -p /usr/src; \
	cd /usr/src; \
	\
	curl -fSL "$PHP_URL" -o php.tar.xz; \
	\
	if [ -n "$PHP_SHA256" ]; then \
		echo "$PHP_SHA256 *php.tar.xz" | sha256sum -c -; \
	fi; \
	if [ -n "$PHP_MD5" ]; then \
		echo "$PHP_MD5 *php.tar.xz" | md5sum -c -; \
	fi; \
	\
	if [ -n "$PHP_ASC_URL" ]; then \
		curl -fSL "$PHP_ASC_URL" -o php.tar.xz.asc; \
		export GNUPGHOME="$(mktemp -d)"; \
		for key in $GPG_KEYS; do \
			gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
		done; \
		gpg --batch --verify php.tar.xz.asc php.tar.xz; \
		command -v gpgconf > /dev/null && gpgconf --kill all; \
		rm -rf "$GNUPGHOME"; \
	fi; \
	\
	apk del --no-network .fetch-deps

COPY docker-php-source /usr/local/bin/

RUN set -xe \
	&& apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		coreutils \
		curl-dev \
		libedit-dev \
		libxml2-dev \
		openssl-dev \
		sqlite-dev \
# start install custom deps
		libpng-dev \
		jpeg-dev \
		freetype-dev \
		bzip2-dev \
		libxslt-dev \
		gettext-dev \
		icu-dev \
		openldap-dev \
		libmcrypt-dev \
		libzip-dev \
		libwebp-dev \
		rabbitmq-c-dev \
# end install custom deps
	\
	&& export CFLAGS="$PHP_CFLAGS" \
		CPPFLAGS="$PHP_CPPFLAGS" \
		LDFLAGS="$PHP_LDFLAGS" \
	&& docker-php-source extract \
	&& cd /usr/src/php \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		\
# make sure invalid --configure-flags are fatal errors intead of just warnings
		--enable-option-checking=fatal \
		\
# https://github.com/docker-library/php/issues/439
		--with-mhash \
		\
# --enable-ftp is included here because ftp_ssl_connect() needs ftp to be compiled statically (see https://github.com/docker-library/php/issues/236)
		--enable-ftp \
# --enable-mbstring is included here because otherwise there's no way to get pecl to use it properly (see https://github.com/docker-library/php/issues/195)
		--enable-mbstring \
# --enable-mysqlnd is included here because it's harder to compile after the fact than extensions are (since it's a plugin for several extensions, not an extension in itself)
		--enable-mysqlnd \
		\
		--with-curl \
		--with-libedit \
		--with-openssl \
		--with-zlib \
# start custom extension
		--enable-sockets \
		--enable-calendar \
		--with-ldap \
		--with-freetype-dir \
		--with-jpeg-dir \
		--with-png-dir \
		--with-gd \
		--with-mysqli \
		--with-pdo-mysql=mysqlnd \
		--enable-shmop \
		--enable-bcmath \
		--enable-sysvmsg \
		--enable-sysvsem \
		--enable-sysvshm \
		--with-mcrypt \
		--enable-pcntl \
		--enable-zip \
		--with-bz2 \
		--enable-soap \
		--enable-exif \
		--enable-intl \
		--with-gettext \
		--enable-opcache \
		--with-xsl \
		--with-xmlrpc \
# end custom extension
		\
# bundled pcre does not support JIT on s390x
# https://manpages.debian.org/stretch/libpcre3-dev/pcrejit.3.en.html#AVAILABILITY_OF_JIT_SUPPORT
		$(test "$gnuArch" = 's390x-linux-gnu' && echo '--without-pcre-jit') \
		\
		$PHP_EXTRA_CONFIGURE_ARGS \
	&& make -j "$(nproc)" \
	&& find -type f -name '*.a' -delete \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; } \
	&& make clean \
	\
# https://github.com/docker-library/php/issues/692 (copy default example "php.ini" files somewhere easily discoverable)
	&& cp -v php.ini-* "$PHP_INI_DIR/" \
	\
# start custom third extension
	&& apk add librdkafka-dev \
	&& cd /tmp \
	&& curl -fSL "https://github.com/ImageMagick/ImageMagick/archive/7.0.8-43.tar.gz" -o ImageMagick-7.0.8-43.tar.gz \
	&& tar zxf ImageMagick-7.0.8-43.tar.gz \
	&& cd ImageMagick-7.0.8-43 \
	&& ./configure --disable-dependency-tracking \
	&& make && make install \
	&& cd /tmp \
	&& curl -fSL "http://pecl.php.net/get/redis-4.3.0.tgz" -o redis-4.3.0.tgz \ 
	&& curl -fSL "http://pecl.php.net/get/mongodb-1.5.3.tgz" -o mongodb-1.5.3.tgz \ 
	&& curl -fSL "http://pecl.php.net/get/rdkafka-3.1.0.tgz" -o rdkafka-3.1.0.tgz \ 
	&& curl -fSL "http://pecl.php.net/get/imagick-3.4.4.tgz" -o imagick-3.4.4.tgz \ 
	&& curl -fSL "http://pecl.php.net/get/amqp-1.9.4.tgz" -o amqp-1.9.4.tgz \ 
	&& cd /tmp && tar zxf redis-4.3.0.tgz && cd redis-4.3.0 \
	&& phpize && ./configure && make && make install \
	&& cd /tmp && tar zxf mongodb-1.5.3.tgz && cd mongodb-1.5.3 \
	&& phpize && ./configure && make && make install \
	&& cd /tmp && tar zxf rdkafka-3.1.0.tgz && cd rdkafka-3.1.0 \
	&& phpize && ./configure && make && make install \
	&& cd /tmp && tar zxf imagick-3.4.4.tgz && cd imagick-3.4.4 \
	&& phpize && ./configure && make && make install \
	&& cd /tmp && tar zxf amqp-1.9.4.tgz && cd amqp-1.9.4 \
	&& phpize && ./configure && make && make install \
	&& cd /tmp \
	&& rm -rf /tmp/redis* \
		/tmp/mongodb* \
		/tmp/rdkafka* \
		/tmp/ImageMagick* \
		/tmp/imagick* \
		/tmp/amqp* \
	&& apk del librdkafka-dev \
	&& echo "zend_extension=opcache.so" > /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
	&& echo "extension=redis.so" > /usr/local/etc/php/conf.d/docker-php-ext-redis.ini \
	&& echo "extension=rdkafka.so" > /usr/local/etc/php/conf.d/docker-php-ext-rdkafka.ini \
	&& echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/docker-php-ext-mongodb.ini \
	&& echo "extension=imagick.so" > /usr/local/etc/php/conf.d/docker-php-ext-imagick.ini \
	&& echo "extension=amqp.so" > /usr/local/etc/php/conf.d/docker-php-ext-amqp.ini \
# end custom third extension
	&& cd / \
	&& docker-php-source delete \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache $runDeps \
	\
	&& apk del --no-network .build-deps \
	\
# https://github.com/docker-library/php/issues/443
	&& pecl update-channels \
	&& rm -rf /tmp/pear ~/.pearrc

COPY docker-php-ext-* /usr/local/bin/

##<autogenerated>##
WORKDIR /var/www/html

RUN set -ex \
	&& cd /usr/local/etc \
	&& if [ -d php-fpm.d ]; then \
		# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
		sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
		cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		# PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
		mkdir php-fpm.d; \
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi \
	&& { \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf

# Add additional binaries into PATH for convenience
ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

# Copy nginx configuration files
COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf
COPY supervisor.conf /etc/supervisord.conf

EXPOSE 80 443

CMD ["/usr/bin/supervisord"]

# Override stop signal to stop process gracefully
# https://github.com/php/php-src/blob/17baa87faddc2550def3ae7314236826bc1b1398/sapi/fpm/php-fpm.8.in#L163
STOPSIGNAL SIGQUIT
