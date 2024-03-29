FROM debian:stretch

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends wget curl ca-certificates build-essential \
                      apt-transport-https gnupg bash-completion

# Sources APT
RUN echo "deb https://packages.sury.org/php/ stretch main" >> /etc/apt/sources.list.d/sury.org.list && \
    wget -O- https://packages.sury.org/php/apt.gpg | apt-key add -

# Install dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      apache2 \
      git \
      sqlite3 \
      php7.2 \
      php7.2-apcu \
      php7.2-curl \
      php7.2-dev \
      php7.2-intl \
      php7.2-json \
      php7.2-mbstring \
      php7.2-opcache \
      php7.2-pgsql \
      php7.2-sqlite3 \
      php7.2-xml \
      php7.2-zip \
      php-pear \
      php-imagick \
      postgresql-client \
      tig \
      unzip \
      vim \
      zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

    # Install composer
ADD install_composer.sh /install_composer.sh
RUN chmod +x /install_composer.sh \
 && sh /install_composer.sh \
 && mv composer.phar /usr/bin/composer \
 && chmod +x /usr/bin/composer

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
ADD php/opcache.ini /etc/php/7.2/mods-available/opcache.ini

RUN mkdir -p /openhanditour \
 && usermod -u 1000 -d /openhanditour -s /bin/bash www-data \
 && chown www-data:www-data /openhanditour

WORKDIR /openhanditour

RUN rm -f /etc/apache2/sites-enabled/000-default.conf

EXPOSE 83

RUN sed -i "s/AllowOverride None/AllowOverride All/g" /etc/apache2/apache2.conf
RUN a2enmod headers
RUN a2enmod rewrite

#HTTP 2
RUN a2enmod http2
RUN a2enmod ssl
ADD http2.conf /etc/apache2/conf-available/http2.conf
RUN a2enconf http2

#Apache conf
ADD apache.vhost.conf /etc/apache2/sites-available/docker-app.conf
RUN a2ensite docker-app

# Change conf
RUN sed -i "s/;date.timezone =/date.timezone = UTC/" /etc/php/7.2/cli/php.ini \
 && sed -i "s/;date.timezone =/date.timezone = UTC/" /etc/php/7.2/apache2/php.ini \
 && sed -i "s/;realpath_cache_size = 4096k/realpath_cache_size = 4096k/" /etc/php/7.2/cli/php.ini \
 && sed -i "s/;realpath_cache_size = 4096k/realpath_cache_size = 4096k/" /etc/php/7.2/apache2/php.ini \
 && sed -i "s/;realpath_cache_ttl = 120/realpath_cache_ttl = 600/" /etc/php/7.2/cli/php.ini \
 && sed -i "s/;realpath_cache_ttl = 120/realpath_cache_ttl = 600/" /etc/php/7.2/apache2/php.ini \
 && sed -i -e "s/^display_errors = Off/display_errors = On/" /etc/php/7.2/apache2/php.ini \
 && sed -i -e "s/^display_errors = Off/display_errors = On/" /etc/php/7.2/cli/php.ini \
 && rm -f /etc/php/7.2/*/conf.d/20-opcache.ini

# Run Apache
ADD run.sh /run.sh

CMD ["/bin/bash", "/run.sh"]
