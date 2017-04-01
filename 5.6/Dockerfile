FROM debian:jessie

MAINTAINER keopx <keopx@keopx.net>

#
# Step 1: Installation
#

# Set frontend. We'll clean this later on!
ENV DEBIAN_FRONTEND noninteractive

# Locale
ENV LOCALE es_ES.UTF-8

# PHP Timezone
ENV TZ=Europe/Madrid

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
  libssh2-php \
  locales

# Install locale
RUN \
  sed -i -e "s/# $LOCALE/$LOCALE/" /etc/locale.gen && \
  echo "LANG=$LOCALE">/etc/default/locale && \
  dpkg-reconfigure --frontend=noninteractive locales && \
  update-locale LANG=$LOCALE

# Install PHP5.6 with Xdebug (dev environment).
RUN apt-get -yqq install \
  php5			\
  php5-curl 		\
  php5-dev 		\
  php5-gd 		\
  php5-intl 		\
  php5-json 		\
  php5-mcrypt		\
  php5-mysql		\
  php5-twig		\
  php5-ssh2		\
  php5-apcu		\
  php5-memcache 	\
  php5-memcached 	\
  php5-redis		\
  php5-xdebug		\
  php5-xhprof		\
  libapache2-mod-php5

# PHP Timezone
RUN \
  echo $TZ | tee /etc/timezone && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  echo "date.timezone = \"$TZ\";" > /etc/php5/apache2/conf.d/timezone.ini && \
  echo "date.timezone = \"$TZ\";" > /etc/php5/cli/conf.d/timezone.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Apache web server.
RUN apt-get -yqq install apache2-mpm-prefork

#
# Step 2: Configuration
#

# Disable by default apcu, xdebug and xhprof. Use docker-compose.yml to add file.
RUN php5dismod apcu xdebug xhprof

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
  php5	 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl			\
  && a2dismod cgi

# Install composer (latest version)
RUN curl -sS https://getcomposer.org/installer | php && \
  mv composer.phar /usr/local/bin/composer && \
  # prestissimo to speed up composer
  composer global require "hirak/prestissimo:^0.3"

### Install DRUSH (latest stable) ###
# Create and/or navigate to a path for the single Composer Drush install.
RUN mkdir --parents /opt/drush-7.x && cd /opt/drush-7.x && \
  # Initialise a new Composer project that requires Drush.
  composer init --require=drush/drush:7.* -n && \
  # Configure the path Composer should use for the Drush vendor binaries.
  composer config bin-dir /usr/local/bin && \
  # Install Drush.
  composer install && \
  # Drush vitamin.
  # See: https://www.drupal.org/project/registry_rebuild.
  drush dl registry_rebuild && \
  # See: https://www.drupal.org/project/updatepath.
  drush dl --destination=$HOME/.drush updatepath -y && drush cache-clear drush

# Bash setup.
RUN echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && \
  echo ". /vendor/drush/drush/drush.complete.sh" >> ~/.bashrc && \
  echo "alias ll='ls -lahs'" >> ~/.bashrc && \
  rm /usr/local/bin/drush.bat /usr/local/bin/drush.complete.sh /usr/local/bin/drush.php

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
