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
  echo "deb http://ftp.de.debian.org/debian/ stretch main non-free contrib" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ stretch main non-free contrib" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
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
  locales \
  libjpeg-turbo-progs libjpeg-progs \
  pngcrush optipng

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
  apt-get -qq update && apt-get -qqy upgrade

# Install PHP7 with Xdebug (dev environment)
RUN apt-get -yqq install \
  php7.2 		\
  php7.2-curl 		\
  php7.2-bcmath   \
  php7.2-bz2   \
  php7.2-dev 		\
  php7.2-gd 		\
  php7.2-dom		\
  php7.2-imap     \
  php7.2-imagick  \
  php7.2-intl 		\
  php7.2-json 		\
  php7.2-ldap 		\
  php7.2-mbstring	\
  php7.2-mysql		\
  php7.2-oauth    \
  php7.2-odbc     \
  php7.2-uploadprogress \
  php7.2-ssh2		\
  php7.2-xml		\
  php7.2-zip		\
  php7.2-solr		\
  php7.2-apcu		\
  php7.2-opcache	\
  php7.2-memcache 	\
  php7.2-memcached 	\
  php7.2-redis		\
  libapache2-mod-php7.2

# php7.2-mcrypt 	\
  
# PHP Timezone
RUN \
  echo $TZ | tee /etc/timezone && \
  dpkg-reconfigure --frontend noninteractive tzdata && \
  echo "date.timezone = \"$TZ\";" > /etc/php/7.2/apache2/conf.d/timezone.ini && \
  echo "date.timezone = \"$TZ\";" > /etc/php/7.2/cli/conf.d/timezone.ini

# PHP xdebug
RUN \
  wget https://github.com/xdebug/xdebug/archive/master.zip && \
  unzip master.zip && \
  cd xdebug-master && \
  phpize7.2 && \
  ./configure --enable-xdebug && \
  make && make install && \
  echo "zend_extension=xdebug.so" >> /etc/php/7.2/mods-available/xdebug.ini && \
  cd .. && rm -fr xdebug-master master.zip

# Install manually xhprof
RUN \
  cd /tmp && \
  wget https://github.com/Yaoguais/phpng-xhprof/archive/master.zip && \
  unzip master.zip && \
  cd phpng-xhprof-master && \
  phpize7.2 && \
  ./configure --with-php-config=/usr/bin/php-config7.2 && \
  make && make install && \
  mv /usr/lib/php/20170718/phpng_xhprof.so /usr/lib/php/20170718/xhprof.so  && \
  echo "extension=xhprof.so" > /etc/php/7.2/mods-available/xhprof.ini && \
  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/7.2/mods-available/xhprof.ini

# Install manually APC
RUN \
  echo "extension=apcu.so" > /etc/php/7.2/mods-available/apcu_bc.ini && \
  echo "extension=apc.so" >> /etc/php/7.2/mods-available/apcu_bc.ini

# Install SMTP.
RUN apt-get install -y ssmtp && \
  echo "FromLineOverride=YES" >> /etc/ssmtp/ssmtp.conf

# Install Apache web server.
RUN apt-get -yqq install apache2

#
# Step 2: Configuration
#

# Enable uploadprogress, imagick, redis and solr.
RUN phpenmod uploadprogress imagick redis solr
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
  php7.2 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl

# Install composer (latest version) | prestissimo to speed up composer
RUN curl -sS https://getcomposer.org/installer | php && \
  mv composer.phar /usr/local/bin/composer && \
  composer global require "hirak/prestissimo:^0.3"

### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest DRUSH project version:
RUN curl http://files.drush.org/drush.phar -L -o drush.phar && \
  mv drush.phar /usr/local/bin/drush && \
  chmod +x /usr/local/bin/drush && \
  drush init -y

### Install DRUPAL CONSOLE (latest version) ###
# Run this in your terminal to get the latest project version:
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
  mv drupal.phar /usr/local/bin/drupal && \
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
