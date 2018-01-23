FROM lxmltj/php7
MAINTAINER Ximing LU <lxmltj@gmail.com>

# Install extra utils
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
