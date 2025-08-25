FROM php:8.2-apache

RUN apt-get update && apt-get install -y \
    unzip curl git \
    libpng-dev libjpeg-dev libldap2-dev libxml2-dev libzip-dev libicu-dev \
 && docker-php-ext-configure gd --with-jpeg \
 && docker-php-ext-install pdo_mysql mysqli gd ldap intl zip opcache exif \
 && rm -rf /var/lib/apt/lists/*

# Instala Composer (para gerar /vendor)
RUN curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php \
 && php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer \
 && rm /tmp/composer-setup.php
ENV COMPOSER_ALLOW_SUPERUSER=1

WORKDIR /var/www/html
# Copia seu fork (branch)
COPY . /var/www/html

# Instala dependências do PHP do GLPI (gera /vendor)
RUN composer install --no-dev --prefer-dist --optimize-autoloader

# Permissões
RUN chown -R www-data:www-data /var/www/html

# Apache → DocumentRoot /public + rewrite + AllowOverride
RUN a2enmod rewrite && \
    sed -ri -e 's!/var/www/html!/var/www/html/public!g' \
      /etc/apache2/sites-available/000-default.conf \
      /etc/apache2/sites-available/default-ssl.conf && \
    printf '<Directory "/var/www/html/public">\n  AllowOverride All\n  Require all granted\n</Directory>\n' \
      > /etc/apache2/conf-available/glpi.conf && a2enconf glpi && \
    echo "ServerName localhost" >> /etc/apache2/apache2.conf

# PHP ini
RUN { echo "upload_max_filesize=64M"; echo "post_max_size=64M"; echo "memory_limit=512M"; echo "max_execution_time=300"; } \
  > /usr/local/etc/php/conf.d/glpi.ini

VOLUME ["/var/www/html/files", "/var/www/html/config", "/var/www/html/marketplace"]
EXPOSE 80
CMD ["apache2-foreground"]
