FROM drupal:10.5-php8.3

RUN apt update && \
    apt install --yes --no-install-recommends \
        mariadb-client git unzip && \
    pecl install \
        apcu && \
    docker-php-ext-enable \
        apcu && \
    rm -rf /var/lib/apt/lists/*

RUN composer require --no-interaction --update-with-dependencies --optimize-autoloader --apcu-autoloader \
        drupal/api drupal/redis drush/drush predis/predis; \
    chown -R www-data:www-data web/modules

RUN mkdir -p /var/cache/php/opcache; \
    echo 'memory_limit = 2G' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini; \
    echo 'error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED' >> /usr/local/etc/php/conf.d/docker-php-error-reporting.ini; \
    echo 'realpath_cache_size = 32M' >> /usr/local/etc/php/conf.d/docker-php-realpath-cache.ini; \
    echo 'realpath_cache_ttl = 3600' >> /usr/local/etc/php/conf.d/docker-php-realpath-cache.ini; \
    echo 'opcache.enable_cli = 0' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \ 
    echo 'opcache.file_cache = /var/cache/php/opcache' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    echo 'opcache.file_cache_consistency_checks = 1' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    echo 'opcache.file_cache_fallback = 1' >> /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    echo 'apc.shm_size = 1G' >> /usr/local/etc/php/conf.d/docker-php-apcu.ini; \
    echo 'apc.enable_cli = 1' >> /usr/local/etc/php/conf.d/docker-php-apcu.ini

COPY drush-site-precreate /usr/local/bin/

ENTRYPOINT ["drush-site-precreate"]