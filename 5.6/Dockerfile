FROM debian:stretch

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
  echo "deb http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ stretch main non-free contrib\n" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ stretch/updates main contrib non-free\n" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
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
  locales

# Install locale
RUN \
  sed -i -e "s/# $LOCALE/$LOCALE/" /etc/locale.gen && \
  echo "LANG=$LOCALE">/etc/default/locale && \
  dpkg-reconfigure --frontend=noninteractive locales && \
  update-locale LANG=$LOCALE

# Configure Sury sources
# @see https://www.noobunbox.net/serveur/auto-hebergement/installer-php-7-1-sous-debian-et-ubuntu
RUN \
  apt-get -yqq install apt-transport-https lsb-release ca-certificates && \
  wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
  echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list && \
  # Update repositories cache and distribution
  apt-get -qq update && apt-get -qqy upgrade

# Install PHP5.6 with Xdebug (dev environment). WARNING with install php5.6-apcu.
RUN apt-get -yqq install \
  php5.6 		\
  php5.6-curl 		\
  php5.6-dev 		\
  php5.6-gd 		\
  php5.6-dom		\
  php5.6-intl 		\
  php5.6-json 		\
  php5.6-mbstring	\
  php5.6-mcrypt 	\
  php5.6-mysql		\
  php5.6-uploadprogress \
  php5.6-ssh2		\
  php5.6-xml		\
  php5.6-zip		\
  php5.6-solr		\
  php5.6-opcache	\
  php5.6-memcache 	\
  php5.6-memcached 	\
  php5.6-redis		\
  php5.6-xdebug		\
  libapache2-mod-php5.6 && \
  apt-get -yqq install php5.6-apcu --no-install-recommends

# PHP Timezone
RUN \
  echo $TZ | tee /etc/timezone && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  echo "date.timezone = \"$TZ\";" > /etc/php/5.6/apache2/conf.d/timezone.ini && \
  echo "date.timezone = \"$TZ\";" > /etc/php/5.6/cli/conf.d/timezone.ini

# Install manually xhprof
RUN \
  pecl install xhprof-beta && \
  echo "extension=xhprof.so" > /etc/php/5.6/mods-available/xhprof.ini && \
  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/5.6/mods-available/xhprof.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Apache web server.
RUN apt-get -yqq install apache2

#
# Step 2: Configuration
#

# Disable by default apcu, opcache, xdebug and xhprof. Use docker-compose.yml to add file.
RUN phpdismod apcu opcache xdebug xhprof

# Remove all sites enabled
RUN rm /etc/apache2/sites-enabled/*

# Configure needed apache modules and disable default site
RUN a2dismod   mpm_event  cgi # mpm_worker enabled.
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
  php5.6 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl			

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
  chmod +x /usr/local/bin/drupal

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
