FROM php:7.4.30-apache-bullseye

ARG APP_VERSION=4.11.0
ENV URL=https://github.com/opensupports/opensupports/releases/download/v${APP_VERSION}/opensupports_v${APP_VERSION}.zip

# Install opensupports
RUN set -ex; \
    apt-get update && apt-get install --yes --no-install-recommends unzip; \
    curl -fsSL -o opensupports.zip $URL; \
    unzip opensupports.zip -d /var/www/html; \
    rm opensupports.zip;

# Fix some permissions
RUN set -ex; \
    a2enmod rewrite; \
    chmod 777 /var/www/html/api/config.php /var/www/html/api/files; \
    chmod -R 777 /var/www/html/api/vendor/ezyang/htmlpurifier/library/HTMLPurifier/DefinitionCache/;

# Install required extensions
RUN set -ex; \
    docker-php-ext-install pdo_mysql; \
    docker-php-ext-install mysqli;

# Install imap extension for email polling suspport
# see also: https://stackoverflow.com/a/38526260/17685342
RUN set -ex; \
    apt-get install --yes --no-install-recommends libc-client-dev libkrb5-dev; \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install imap;

# Fix reverse proxy using https
COPY fix-https-reverse-proxy.diff /var/www/html/
RUN patch /var/www/html/index.php < /var/www/html/fix-https-reverse-proxy.diff;

# Fix email polling 'undefined method' error
# see also: https://github.com/opensupports/opensupports/pull/1154
COPY fix-email-polling.patch /var/www/html/
RUN patch /var/www/html/api/controllers/system/email-polling.php < /var/www/html/fix-email-polling.patch

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
