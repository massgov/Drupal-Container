
FROM php:8.0-apache

ENV DEBIAN_FRONTEND noninteractive
ENV APACHE_DOCROOT /var/www/html
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_REQUEST_WORKERS 150
ENV MEMORY_LIMIT 256m
ENV PATH=/var/www/vendor/bin:$PATH

RUN apt-get update && apt-get install -y --no-install-recommends \
      # php7.3+ needs zlib1g-dev and libzip-dev for the zip extension
      # https://github.com/docker-library/php/issues/61#issuecomment-468874705
      zlib1g-dev \
      libzip-dev \
      #for life
      unzip \
      # for bz2
      bzip2 libbz2-dev \
      # for ftp
      libssl-dev \
      # for gd
      libfreetype6-dev libjpeg62-turbo-dev libpng-dev \
      # for intl
      libicu-dev \
      # for dom
      libxml2-dev \
      # for mcrypt
      libmcrypt-dev \
      # for memcached
      libz-dev libmemcached-dev \
      # for mysql
      mariadb-client \
      nano \
      # for drush ma:deploy
      wget \
      # for drush sql:sync
      rsync \
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

RUN docker-php-ext-configure gd  \
    && docker-php-ext-install -j$(nproc) \
      bcmath \
      bz2 \
      calendar \
      exif \
      ftp \
      gd \
      gettext \
      intl \
      mysqli \
      opcache \
      pcntl \
      pdo_mysql \
      shmop \
      soap \
      sysvmsg \
      sysvsem \
      sysvshm \
      zip \
    && pecl install apcu memcached raphf yaml imagick \
    && docker-php-ext-enable apcu memcached raphf yaml imagick \
    && printf "upload_max_filesize = 128M\npost_max_size = 128M" > $PHP_INI_DIR/conf.d/00-max_filesize.ini

#https://github.com/docker-library/php/issues/1245#issuecomment-1020294919
RUN CFLAGS="$CFLAGS -D_GNU_SOURCE" docker-php-ext-install sockets

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
  && apt-get install -y apt-transport-https lsb-release gnupg > /dev/null 2>&1 \
  && curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
  && echo 'deb https://deb.nodesource.com/node_10.x jessie main' > /etc/apt/sources.list.d/nodesource.list \
  && echo 'deb-src https://deb.nodesource.com/node_10.x jessie main' >> /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install -y nodejs \
  && apt-get install -y npm \
  && npm install -g yarn \
  && npm cache clean --force \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# CI CLI tools:
RUN apt-get update \
  && apt-get -y -qq install awscli \
  && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*