FROM debian:buster

MAINTAINER keopx <keopx@keopx.net>

#
# Step 1: Installation
#

# System environments
ENV DEBIAN_FRONTEND="noninteractive" \
    LOCALE="es_ES.UTF-8" \
    GOTPL_VER="0.1.5" \
    DEFAULT_ROOT="/var/www/html" \
    UID="1000" \
    GID="1000" \
    UNAME="keopx"

# Set repositories
RUN \
  echo "deb http://ftp.de.debian.org/debian/ buster main non-free contrib" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ buster main non-free contrib" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ buster/updates main contrib non-free" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ buster/updates main contrib non-free" >> /etc/apt/sources.list && \
  apt-get -qq update && apt-get -qqy upgrade && \
# Install some basic tools needed for deployment
  apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  default-mysql-client \
  locales \
  curl \
  wget \
  unzip \
  patch \
  rsync \
  vim \
  nano \
  openssh-client \
  git \
  bash-completion \
  locales \
  libjpeg-turbo-progs libjpeg-progs \
  pngcrush optipng && \
# Install locale
  sed -i -e "s/# $LOCALE/$LOCALE/" /etc/locale.gen && \
  echo "LANG=$LOCALE">/etc/default/locale && \
  dpkg-reconfigure --frontend=noninteractive locales && \
  update-locale LANG=$LOCALE && \
# GOTPL
  gotpl_url="https://github.com/wodby/gotpl/releases/download/${GOTPL_VER}/gotpl-linux-amd64-${GOTPL_VER}.tar.gz"; \
  wget -qO- "${gotpl_url}" | tar xz -C /usr/local/bin; \
# Configure Sury sources
# @see https://www.noobunbox.net/serveur/auto-hebergement/installer-php-7-1-sous-debian-et-ubuntu
  apt-get -yqq install apt-transport-https lsb-release ca-certificates && \
  wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
  echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list && \
# Install PHP7 with Xdebug (dev environment)
  apt-get -qq update && \
  apt-get -yqq install \
  php8.0 		\
  php8.0-bcmath   \
  php8.0-bz2   \
  php8.0-curl 		\
  php8.0-dev 		\
  php8.0-gd 		\
  php8.0-dom		\
  php8.0-imap     \
  php8.0-intl 		\
  php8.0-ldap 		\
  php8.0-mbstring	\
  php8.0-mysql		\
  php8.0-oauth		\
  php8.0-odbc		\
  php8.0-xml		\
  php8.0-yaml		\
  php8.0-zip		\
  php8.0-solr		\
  php8.0-apcu		\
  php8.0-opcache	\
  php8.0-redis		\
  php8.0-memcache 	\
  php8.0-xdebug		\
  libapache2-mod-php8.0 && \
# Install manually xhprof
#  cd /tmp && \
#  wget https://github.com/Yaoguais/phpng-xhprof/archive/master.zip && \
#  unzip master.zip && \
#  cd phpng-xhprof-master && \
#  phpize8.0 && \
#  ./configure --with-php-config=/usr/bin/php-config8.0 && \
#  make && make install && \
#  mv /usr/lib/php/20190902/phpng_xhprof.so /usr/lib/php/20190902/xhprof.so  && \
#  echo "extension=xhprof.so" > /etc/php/8.0/mods-available/xhprof.ini && \
#  echo "xhprof.output_dir=/var/www/xhprof" >> /etc/php/8.0/mods-available/xhprof.ini && \

# Install manually APC
#  echo "extension=apcu.so" > /etc/php/7.2/mods-available/apcu_bc.ini && \
#  echo "extension=apc.so" >> /etc/php/7.2/mods-available/apcu_bc.ini && \
# Install SMTP.
  apt-get -yqq install libgnutls-openssl27 && \
  wget ftp.de.debian.org/debian/pool/main/s/ssmtp/ssmtp_2.64-8+b2_amd64.deb && \
  dpkg -i ssmtp_2.64-8+b2_amd64.deb && rm ssmtp_2.64-8+b2_amd64.deb && \
# Install Apache web server.
  apt-get -yqq install apache2 && \
#
# Step 2: Configuration
#
# Enable uploadprogress, imagick, redis and solr.
  phpenmod uploadprogress imagick redis solr && \
# Disable by default apcu, apcu_bc, opcache, xdebug and xhprof. Use docker-compose.yml to add file.
  phpdismod apcu apcu_bc opcache xdebug xhprof && \
# Remove all sites enabled
# RUN rm /etc/apache2/sites-enabled/*
# Configure needed apache modules and disable default site
# mpm_worker enabled.
  a2dismod mpm_event cgi && \
  a2enmod		\
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
  php8.0 		\
  mpm_prefork 		\
  reqtimeout 		\
  rewrite 		\
  setenvif 		\
  status 		\
  ssl && \
# without the following line we get "AH00558: apache2: Could not reliably determine the server's fully qualified domain name"
# autorise .htaccess files
  sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf && \
# Install composer (latest version) | prestissimo to speed up composer
  php -r "copy('https://getcomposer.org/composer.phar', 'composer.phar');"  && \
  php composer.phar  && \
  mv composer.phar /usr/local/bin/composer && \
  chmod 755 /usr/local/bin/composer && \
### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest DRUSH project version:
  composer global require drush/drush && \
  composer global require symfony/http-kernel -W  && \
  curl -OL https://github.com/drush-ops/drush-launcher/releases/download/0.6.0/drush.phar && \
  mv drush.phar /usr/local/bin/drush && \
  chmod +x /usr/local/bin/drush && \
# Bash setup.
  echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && echo "alias ll='ls -lahs'" >> ~/.bashrc && \
#
# Step 3: Clean the system
#
# Cleanup some things.
  apt-get -q autoclean && \
  rm -rf /var/lib/apt/lists/* && \
#
# Step 4: Run
#
# Create 'keopx' user like local machime user.
  groupadd -g $UID $GID ; \
  useradd -m -u $UID -g $GID -s /bin/bash $UNAME ; \
  usermod -aG www-data $UNAME; \
  echo ". /usr/share/bash-completion/bash_completion" >> ~/.bashrc && echo "alias ll='ls -lahs'" >> /home/$UNAME/.bashrc
# Working dir
WORKDIR ${DEFAULT_ROOT}
# Configure templates
COPY templates /etc/gotpl/
COPY scripts/apache2-foreground /usr/bin/
EXPOSE 80 443
CMD ["apache2-foreground"]
