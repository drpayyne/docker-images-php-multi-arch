ARG PHP_TAG
FROM ghcr.io/drpayyne/php-fpm:${PHP_TAG}

# Install and enable SourceGuardian and ionCube loaders
RUN install-php-extensions sourceguardian ioncube_loader
