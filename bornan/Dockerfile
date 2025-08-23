FROM php:8.2-apache

# Dependências exigidas/recomendadas pelo GLPI
# (gd, intl, mysqli/pdo_mysql, zip, exif, ldap, opcache etc.)
RUN apt-get update && apt-get install -y \
    unzip curl git \
    libpng-dev libjpeg-dev libldap2-dev libxml2-dev libzip-dev libicu-dev \
    && docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install pdo_mysql mysqli gd ldap intl zip opcache exif \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html

# Copia o GLPI do seu fork (o checkout é feito no CI)
COPY . /var/www/html

# Permissões
RUN chown -R www-data:www-data /var/www/html

# Apache: DocumentRoot -> /public, AllowOverride e rewrite
RUN a2enmod rewrite && \
    sed -ri -e 's!/var/www/html!/var/www/html/public!g' \
      /etc/apache2/sites-available/000-default.conf \
      /etc/apache2/sites-available/default-ssl.conf && \
    printf '<Directory "/var/www/html/public">\n  AllowOverride All\n  Require all granted\n</Directory>\n' \
      > /etc/apache2/conf-available/glpi.conf && \
    a2enconf glpi && \
    echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Ajustes básicos de PHP
RUN { \
      echo "upload_max_filesize=64M"; \
      echo "post_max_size=64M"; \
      echo "memory_limit=512M"; \
      echo "max_execution_time=300"; \
    } > /usr/local/etc/php/conf.d/glpi.ini

# Volumes que serão montados no AKS
VOLUME ["/var/www/html/files", "/var/www/html/config", "/var/www/html/marketplace"]

EXPOSE 80
CMD ["apache2-foreground"]
