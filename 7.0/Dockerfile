FROM debian:jessie

MAINTAINER keopx <keopx@keopx.net>

#
# Step 1: Installation
#

# Set frontend. We'll clean this later on!
ENV DEBIAN_FRONTEND noninteractive

ENV LOCALE es_ES.UTF-8

# Set repositories
RUN \
  echo "deb http://ftp.de.debian.org/debian/ jessie main non-free contrib\n" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ jessie main non-free contrib\n" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ jessie/updates main contrib non-free\n" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ jessie/updates main contrib non-free" >> /etc/apt/sources.list && \
  # Update repositories cache and distribution
  apt-get -qq update && apt-get -qqy upgrade

# Install some basic tools needed for deployment
RUN apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  mysql-client \
  locales \
  curl \
  wget \  
  unzip \
  patch \
  rsync \
  vim \
  openssh-client \
  git \
  bash-completion \
  libssh2-php

# Configure Dotdeb sources
RUN \
  wget -O - http://www.dotdeb.org/dotdeb.gpg | apt-key add - && \
  echo "deb http://packages.dotdeb.org jessie all\n" > /etc/apt/sources.list.d/dotdeb.list && \
  echo "deb-src http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list.d/dotdeb.list && \
  apt-get -qq update && apt-get -qqy upgrade

# Install PHP7 with Xdebug (dev environment)
RUN apt-get -yqq install \
  php7.0 		\
  php7.0-curl 		\
  php7.0-dev 		\
  php7.0-gd 		\
  php7.0-dom		\
  php7.0-intl 		\
  php7.0-json 		\
  php7.0-mbstring	\
  php7.0-mcrypt 	\
  php7.0-mysql		\
  php7.0-ssh2		\
  php7.0-xml		\
  php7.0-zip		\
  php7.0-apcu		\
  php7.0-opcache	\
  php7.0-memcached 	\
  php7.0-redis		\
  php7.0-xdebug		\
  libapache2-mod-php7.0

# Install manually xhprof
RUN \
  cd /tmp && \
  wget https://github.com/RustJason/xhprof/archive/php7.zip && \
  unzip php7.zip && \
  cd xhprof-php7/extension && \
  phpize7.0  && \
  ./configure --with-php-config=/usr/bin/php-config7.0  && \
  make && make install && \
  echo "extension=xhprof.so" > /etc/php/7.0/mods-available/xhprof.ini && \
  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/7.0/mods-available/xhprof.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Apache web server.
RUN apt-get -yqq install apache2-mpm-prefork

#
# Step 2: Configuration
#

# Disable by default xdebug. Use docker-compose to add file.
RUN phpdismod xdebug

# Remove all sites enabled
RUN rm /etc/apache2/sites-enabled/*

# Configure needed apache modules and disable default site
RUN a2enmod		\
  access_compat		\
  actions		\
  alias			\
  auth_basic		\
  authn_core		\
  authn_file		\
  authz_core		\
  authz_groupfile	\
  authz_host 		\
  authz_user		\
  autoindex		\
  dir			\
  env 			\
  expires 		\
  filter 		\
  headers		\
  mime 			\
  negotiation 		\
  php7.0 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  && a2dismod cgi
  
# Install composer (latest version)
RUN curl -sS https://getcomposer.org/installer | php && \
  mv composer.phar /usr/local/bin/composer && \
  # prestissimo to speed up composer
  composer global require "hirak/prestissimo:^0.3"

### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest DRUSH project version:
RUN curl http://files.drush.org/drush.phar -L -o drush.phar && \
  # Accessing from anywhere on your system:
  mv drush.phar /usr/local/bin/drush && \
  # Apply executable permissions on the downloaded file:
  chmod +x /usr/local/bin/drush && \
  # Copy configuration files to user home directory:
  drush init -y

### Install DRUPAL CONSOLE (latest version) ###
# Run this in your terminal to get the latest project version:
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
  # Accessing from anywhere on your system:
  mv drupal.phar /usr/local/bin/drupal && \
  # Apply executable permissions on the downloaded file:
  chmod +x /usr/local/bin/drupal && \
  # Copy configuration files to user home directory:
  drupal init --override

# Bash setup.
RUN echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && echo "alias ll='ls -lahs'" >> ~/.bashrc

#
# Step 3: Clean the system
#

# Cleanup some things.
RUN apt-get -q autoclean && \
  rm -rf /var/lib/apt/lists/*

#
# Step 4: Run
#

# Create 'me' user like local machime user.
RUN useradd me && usermod -G www-data -a me

# Working dir
WORKDIR /var/www

# Volume for Apache2 data
VOLUME /var/www

COPY scripts/apache2-foreground /usr/bin/

EXPOSE 80 443

CMD ["apache2-foreground"]
