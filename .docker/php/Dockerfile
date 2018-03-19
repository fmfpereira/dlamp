# SET DEFAULT PHP VERSION
ARG DOCKER_PHP_VERSION=7

# USE ARG PHP_VERSION
FROM php:${DOCKER_PHP_VERSION}-apache
MAINTAINER fmfpereira@gmail.com

# SHOW CURRENT PHP VERSION
RUN set -xe; echo "USING PHP $PHP_VERSION"

# RUN UPDATES WITH APT-GET
RUN  apt-get update \
  && apt-get upgrade -y \
  && apt-get autoremove -y \
  && apt-get install -yq \
    libmcrypt-dev \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libicu-dev \
    libxml2-dev \
    libxslt-dev \
    mysql-client \
    curl \
    git \
    vim \
    nano \
    && rm -rf /var/lib/apt/lists/*
    
# ENABLE GD WITH JPEG
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/

# INSTALL PHP EXTENSIONS
RUN docker-php-ext-install gd json mbstring pdo pdo_mysql mysqli zip opcache intl xsl
    
# INSTALL XDEBUG
RUN pecl install xdebug

# SET XDEBUG CONFIGURATION
RUN echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=off\n" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_port=9000\n" >> /usr/local/etc/php/conf.d/xdebug.ini

# VALIDATE AND INSTALL COMPOSER
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && php -r "copy('https://composer.github.io/installer.sig', 'installer.sig');" \
  && php -r "if (hash_file('SHA384', 'composer-setup.php') === trim(file_get_contents('installer.sig'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); unlink('installer.sig');} echo PHP_EOL;" \
  && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
  && php -r "unlink('composer-setup.php'); unlink('installer.sig');"

# BUILD ARGUMENTS
ARG DOCKER_APACHE_RUN_USER=www-data
ARG DOCKER_APACHE_RUN_GROUP=www-data
ARG DOCKER_APACHE_RUN_UID=33
ARG DOCKER_APACHE_RUN_GID=33

# ENVIRONMENT VARIABLES
ENV APACHE_RUN_USER=$DOCKER_APACHE_RUN_USER
ENV APACHE_RUN_GROUP=$DOCKER_APACHE_RUN_GROUP
ENV APACHE_RUN_UID=$DOCKER_APACHE_RUN_UID
ENV APACHE_RUN_GID=$DOCKER_APACHE_RUN_GID
ENV PATH="/var/www/html/vendor/bin:${PATH}"

# CREATE USER IF NEW AND SET AS APACHE USER AND GROUP
RUN set -xe; \
  \
  if [ -z $(getent group $APACHE_RUN_GROUP) ]; then \
    echo "Creating group $APACHE_RUN_GROUP"; \
    groupadd --gid $APACHE_RUN_GID $APACHE_RUN_GROUP; \
  else \
    echo "Ignoring group $APACHE_RUN_GROUP: already exists"; \
  fi; \
  \
  if [ -z $(getent passwd $APACHE_RUN_USER) ]; then \
    echo "Creating user $APACHE_RUN_USER"; \
    useradd $APACHE_RUN_USER --uid $APACHE_RUN_UID --gid $APACHE_RUN_GROUP --create-home --shell /bin/bash; \
  else \
    echo "Ignoring user $APACHE_RUN_USER: already exists"; \
  fi;

# ENABLE APACHE REWRITE MODULE 
RUN a2enmod rewrite

