FROM drupal:10.5-php8.3

RUN apt update && apt install --yes --no-install-recommends postgresql-client git unzip && rm -rf /var/lib/apt/lists/*

RUN echo 'memory_limit = 2048M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini; \
    echo 'error_reporting = E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED' >> /usr/local/etc/php/conf.d/docker-php-error-reporting.ini

RUN composer require --no-interaction drupal/api drush/drush && chown -R www-data:www-data web/modules

COPY drush-site-precreate wait-for-postgres /usr/local/bin/

ENTRYPOINT ["drush-site-precreate"]