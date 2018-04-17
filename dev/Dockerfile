
FROM php:7.1-apache

ENV DEBIAN_FRONTEND noninteractive
ENV APACHE_DOCROOT /var/www/html
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_REQUEST_WORKERS 150
ENV SENDMAIL_PATH /bin/true
ENV PATH=/var/www/mass.local/vendor/bin:$PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
      # for bz2
      bzip2 libbz2-dev \
      # for ftp
      libssl-dev \
      # for gd
      libfreetype6-dev libjpeg62-turbo-dev libpng12-dev \
      # for intl
      libicu-dev \
      # for dom
      libxml2-dev \
      # for ldap
      libldap2-dev \
      # for mcrypt
      libmcrypt-dev \
      # for mysql
      mysql-client \
      # for git
      git \
      # for ssh client only
      openssh-client \
      # For image optimization
      jpegoptim \
      optipng \
      pngquant \
      # For imagick
      imagemagick \
      libmagickwand-dev \
      # for yaml
      libyaml-dev \
    && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
    && docker-php-ext-install -j$(nproc) \
      bcmath \
      bz2 \
      calendar \
      exif \
      ftp \
      gd \
      gettext \
      intl \
      ldap \
      mcrypt \
      mysqli \
      opcache \
      pcntl \
      pdo_mysql \
      shmop \
      soap \
      sockets \
      sysvmsg \
      sysvsem \
      sysvshm \
      zip \
    && pecl install apcu raphf propro yaml imagick \
    && docker-php-ext-enable apcu raphf propro yaml imagick \
    && pecl install pecl_http \
    && docker-php-ext-enable http \
    && printf "upload_max_filesize = 128M\npost_max_size = 128M" > $PHP_INI_DIR/conf.d/00-max_filesize.ini

# Configure Apache:
RUN a2enmod rewrite headers expires \
  && sed -i "/User www-data/c\User \$\{APACHE_RUN_USER\}" /etc/apache2/apache2.conf \
  && sed -i "/Group www-data/c\Group \$\{APACHE_RUN_GROUP\}" /etc/apache2/apache2.conf \
  && sed -i "/DocumentRoot \/var\/www\/html/c\\\tDocumentRoot \$\{APACHE_DOCROOT\}" /etc/apache2/sites-enabled/000-default.conf \
  # Preemptively add a user 1000, for use with $APACHE_RUN_USER on osx
  && adduser --uid 1000 --gecos 'My OSX User' --disabled-password osxuser

# Install CLI tools:
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/bin/composer \
  && mkdir /usr/share/composer \
  && cd /usr/share/composer \
  && composer init --no-interaction && composer config bin-dir /usr/local/bin \
  && rm -r /root/.composer/cache \
  && curl -L https://github.com/jwilder/dockerize/releases/download/v0.6.0/dockerize-linux-amd64-v0.6.0.tar.gz | tar -C /usr/local/bin -xzv \
  && chmod +x /usr/local/bin/dockerize \
  && ssh-keyscan github.com >> /etc/ssh/ssh_known_hosts \
  && git config --global core.safecrlf false

ADD etc/mpm_prefork.conf /etc/apache2/mods-enabled/mpm_prefork.conf
ADD etc/php-overrides.ini $PHP_INI_DIR/conf.d/zz-overrides.ini
ADD bin/* /usr/local/bin/
CMD ["apache2-foreground-enhanced"]

# NodeJS,
RUN apt-get update \
  && apt-get install -y apt-transport-https lsb-release > /dev/null 2>&1 \
  && curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && echo 'deb https://deb.nodesource.com/node_6.x jessie main' > /etc/apt/sources.list.d/nodesource.list \
  && echo 'deb-src https://deb.nodesource.com/node_6.x jessie main' >> /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install nodejs \
  && npm install -g yarn \
  && npm cache clean --force \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install the Sendmail alternative suggested by Mailhog.
RUN curl -O https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
  && mv mhsendmail_linux_amd64 /usr/bin/mhsendmail \
  && chmod +x /usr/bin/mhsendmail


# Install development tools.
RUN pecl install xdebug \
  && docker-php-ext-enable xdebug \
  && echo "xdebug.remote_enable=on\nxdebug.remote_host=docker.for.mac.host.internal\nxdebug.remote_autostart=on\nxdebug.idekey=PHPSTORM" > $PHP_INI_DIR/conf.d/xdebug.ini \
  # Install Blackfire Probe
  && version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
  && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
  && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
  && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
  && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini \
  # Install blackfire CLI
  && mkdir -p /tmp/blackfire \
  && curl -A "Docker" -L https://blackfire.io/api/v1/releases/client/linux_static/amd64 | tar zxp -C /tmp/blackfire \
  && mv /tmp/blackfire/blackfire /usr/bin/blackfire \
  && rm -Rf /tmp/blackfire

