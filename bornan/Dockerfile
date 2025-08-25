FROM php:8.2-apache

# Dependências de sistema
RUN apt-get update && apt-get install -y \
    unzip curl git \
    libpng-dev libjpeg-dev libldap2-dev libxml2-dev libzip-dev libicu-dev \
 && rm -rf /var/lib/apt/lists/*

# Extensões PHP exigidas/recomendadas pelo GLPI
RUN docker-php-ext-configure gd --with-jpeg \
 && docker-php-ext-install \
    pdo_mysql \
    mysqli \
    gd \
    ldap \
    intl \
    zip \
    opcache \
    exif \
    bcmath    # <— **novo**: requerimento do seu composer.lock

# Composer
RUN curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php \
 && php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer \
 && rm /tmp/composer-setup.php
ENV COMPOSER_ALLOW_SUPERUSER=1

WORKDIR /var/www/html

# Otimiza cache: instala deps PHP primeiro
COPY composer.json composer.lock ./
RUN composer install --no-dev --prefer-dist --optimize-autoloader --no-interaction --no-ansi

# Copia o restante do código do seu fork
COPY . /var/www/html

# Permissões
RUN chown -R www-data:www-data /var/www/html

# Apache -> DocumentRoot /public + AllowOverride + rewrite
RUN a2enmod rewrite && \
    sed -ri -e 's!/var/www/html!/var/www/html/public!g' \
      /etc/apache2/sites-available/000-default.conf \
      /etc/apache2/sites-available/default-ssl.conf && \
    printf '<Directory "/var/www/html/public">\n  AllowOverride All\n  Require all granted\n</Directory>\n' \
      > /etc/apache2/conf-available/glpi.conf && \
    a2enconf glpi && \
    echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Ajustes PHP básicos
RUN { \
      echo "upload_max_filesize=64M"; \
      echo "post_max_size=64M"; \
      echo "memory_limit=512M"; \
      echo "max_execution_time=300"; \
    } > /usr/local/etc/php/conf.d/glpi.ini

# Volumes para AKS
VOLUME ["/var/www/html/files", "/var/www/html/config", "/var/www/html/marketplace"]

EXPOSE 80
CMD ["apache2-foreground"]
