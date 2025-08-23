# Base oficial PHP + Apache
FROM php:8.2-apache

# Instala dependências necessárias
RUN apt-get update && apt-get install -y \
    unzip \
    curl \
    git \
    libpng-dev \
    libjpeg-dev \
    libldap2-dev \
    libxml2-dev \
    libzip-dev \
    libicu-dev \
    cron \
    && docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install \
        pdo_mysql \
        mysqli \
        gd \
        ldap \
        intl \
        zip \
        opcache \
        exif \
    && rm -rf /var/lib/apt/lists/*

# Define diretório de trabalho do Apache
WORKDIR /var/www/html

# Copia o GLPI do seu fork para dentro do container
# (no pipeline de CI/CD o checkout do código já estará disponível)
COPY . /var/www/html

# Permissões adequadas para o Apache
RUN chown -R www-data:www-data /var/www/html

# Configurações PHP básicas
RUN a2enmod rewrite && \
    echo "upload_max_filesize = 64M" > /usr/local/etc/php/conf.d/uploads.ini && \
    echo "post_max_size = 64M" >> /usr/local/etc/php/conf.d/uploads.ini && \
    echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/uploads.ini && \
    echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/uploads.ini

# Define volumes para persistência em AKS
VOLUME ["/var/www/html/files", "/var/www/html/config", "/var/www/html/marketplace"]

EXPOSE 80

CMD ["apache2-foreground"]
