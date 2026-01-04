# PrestaShop Dockerfile for Render
FROM php:8.1-apache

# Install required PHP extensions
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libzip-dev \
    libxml2-dev \
    libicu-dev \
    unzip \
    && docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install \
        pdo_mysql \
        mysqli \
        gd \
        zip \
        intl \
        mbstring \
        xml \
        bcmath \
        opcache \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
COPY --from=composer:2.6 /usr/bin/composer /usr/bin/composer

# Configure Apache
RUN a2enmod rewrite
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

WORKDIR /var/www/html

# Copy and install dependencies
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Copy application code
COPY . .

# Copy database initialization script
COPY init-db.sql /docker-entrypoint-initdb.d/

# Create pre-configured parameters.php to skip installation
RUN mkdir -p /var/www/html/app/config && \
    echo '<?php\n\
return [\n\
    "parameters" => [\n\
        "database_host" => "mysql",\n\
        "database_port" => "3306",\n\
        "database_name" => "prestashop",\n\
        "database_user" => "root",\n\
        "database_password" => getenv("MYSQL_ROOT_PASSWORD") ?: "prestashop",\n\
        "database_prefix" => "ps_",\n\
        "database_engine" => "InnoDB",\n\
        "cookie_key" => "'$(openssl rand -hex 32)'",\n\
        "cookie_iv" => "'$(openssl rand -hex 8)'",\n\
        "new_cookie_key" => "'$(openssl rand -hex 32)'",\n\
        "ps_caching" => "CacheMemcache",\n\
        "ps_cache_enable" => true,\n\
        "ps_creation_date" => "'$(date +%Y-%m-%d)'",\n\
        "secret" => "'$(openssl rand -hex 32)'",\n\
        "mailer_transport" => "smtp",\n\
        "mailer_host" => "127.0.0.1",\n\
        "mailer_user" => null,\n\
        "mailer_password" => null,\n\
    ],\n\
];' > /var/www/html/app/config/parameters.php

# Create .htaccess to redirect /install to /admin
RUN echo 'RewriteEngine On\n\
RewriteRule ^install/?$ admin/ [R=302,L]' >> /var/www/html/.htaccess

# Set permissions
RUN chown -R www-data:www-data /var/www/html
RUN chmod -R 755 /var/www/html
RUN chmod -R 777 /var/www/html/var/cache /var/www/html/var/logs /var/www/html/img /var/www/html/download /var/www/html/upload 2>/dev/null || true

EXPOSE 80

CMD ["apache2-foreground"]
