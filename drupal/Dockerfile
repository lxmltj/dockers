# based on https://hub.docker.com/_/drupal/
# from https://www.drupal.org/requirements/php#drupalversions
FROM php:7.1-apache
MAINTAINER Ximing LU <lxmltj@gmail.com>

# Changing DocumentRoot
RUN  if [ "$APACHE_DOCUMENT_ROOT" != "/var/www/html" ]; \
     then sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf \
     && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf; \
     fi

RUN a2enmod rewrite

# install the PHP extensions we need
RUN set -ex \
	&& buildDeps=' \
		libjpeg62-turbo-dev \
		libpng12-dev \
		libpq-dev \
	' \
	&& apt-get update && apt-get install -y --no-install-recommends $buildDeps && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure gd \
		--with-jpeg-dir=/usr \
		--with-png-dir=/usr \
	&& docker-php-ext-install -j "$(nproc)" gd mbstring opcache pdo pdo_mysql pdo_pgsql zip \
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/gd.so' - libjpeg.so.62: cannot open shared object file: No such file or directory in Unknown on line 0
# PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/local/lib/php/extensions/no-debug-non-zts-20151012/pdo_pgsql.so' - libpq.so.5: cannot open shared object file: No such file or directory in Unknown on line 0
	&& apt-mark manual \
		libjpeg62-turbo \
		libpq5 \
	&& apt-get purge -y --auto-remove $buildDeps

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Sets extra php.ini settings.
RUN { \
		echo 'date.timezone = Europe/Paris'; \
		echo 'upload_max_filesize = 256M'; \
		echo 'post_max_size = 256M'; \
		echo 'xdebug.max_nesting_level = 256'; \
		echo 'memory_limit = 256M'; \
	} > /usr/local/etc/php/conf.d/custom-settings.ini

# Install extra utils
# Add eslint csslint bower grunt
RUN  apt-get update && apt-get install -y nodejs npm
# Symlinks node directory.
RUN ln -s /usr/bin/nodejs /usr/bin/node
# Installs node modules.
RUN npm install -g \
	  eslint \
	  csslint \
	  bower \
	  grunt-cli

# Add composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && chmod a+x /usr/local/bin/composer

# Add drush
RUN php -r "readfile('http://files.drush.org/drush.phar');" > drush \
    && chmod +x drush \
    && mv drush /usr/local/bin/

# Add drupal console
RUN curl https://drupalconsole.com/installer -L -o drupal.phar \
	&& php -r "readfile('https://drupalconsole.com/installer');" > drupal.phar \
	&& mv drupal.phar /usr/local/bin/drupal \
	&& chmod +x /usr/local/bin/drupal \
	&& drupal init --override

# Clean Up
RUN apt-get -y autoremove
RUN apt-get -y clean
