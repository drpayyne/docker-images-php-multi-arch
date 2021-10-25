ARG PHP_TAG
FROM php:${PHP_TAG}

# Refresh apt
RUN set -eux \
    && apt update -y \
    && apt upgrade -y

# Install PHP extension installer
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

# List of PHP extensions
ARG PHP_EXTENSIONS="amqp bcmath gd gmp imagick intl msgpack opcache pdo_mysql redis soap sockets xmlrpc xsl zip"

# Install PHP extensions and other utilities
RUN chmod +x /usr/local/bin/install-php-extensions && sync \
	&& install-php-extensions ${PHP_EXTENSIONS} \
	&& apt update -y \
	&& apt install -y git npm gettext unzip pwgen nmap ncat jq

# Set primary INI file and PHP memory limit
RUN set -eux \
    && cp ${PHP_INI_DIR}/php.ini-development ${PHP_INI_DIR}/php.ini \
    && perl -pi -e 's/memory_limit(.*)/memory_limit=-1/g' ${PHP_INI_DIR}/php.ini

# Setup Composer 1 & 2
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /tmp/composer
COPY --from=composer:1 /usr/bin/composer /usr/bin/composer
COPY --from=composer:1 /usr/bin/composer /usr/bin/composer1
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer2

RUN set -eux; \
	cd /usr/local/etc/; \
	{ \
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
	} | tee php-fpm.d/docker.conf; \
	{ \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf; \
    perl -pi -e 's/^(pid|error_log|daemonize)/;$1/g' php-fpm.conf; \
    perl -pi -e 's/^(listen)/;$1/g' php-fpm.d/www.conf; \
    perl -pi -e 's/^(php_admin_(value|flag))/;$1/g' php-fpm.d/www.conf

# Configure Docker entrypoint
COPY docker-entrypoint /usr/local/bin
ENTRYPOINT ["docker-entrypoint"]

# Override stop signal to stop process gracefully
# https://github.com/php/php-src/blob/17baa87faddc2550def3ae7314236826bc1b1398/sapi/fpm/php-fpm.8.in#L163
STOPSIGNAL SIGQUIT

# Finish up
EXPOSE 9000
CMD ["php-fpm"]
