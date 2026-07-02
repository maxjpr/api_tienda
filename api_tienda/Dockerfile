# Imagen base PHP 8.3 con Apache
FROM php:8.3-apache

# Instalar extensiones PHP necesarias para Laravel
RUN apt-get update && apt-get install -y \
    git \
    curl \
    zip \
    unzip \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libsqlite3-dev \
    && docker-php-ext-install pdo pdo_sqlite mbstring exif pcntl bcmath gd \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Instalar Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configurar Apache: habilitar mod_rewrite y apuntar al public/
RUN a2enmod rewrite
RUN sed -i 's|/var/www/html|/var/www/html/public|g' /etc/apache2/sites-available/000-default.conf
RUN echo '<Directory /var/www/html/public>\n    AllowOverride All\n    Require all granted\n</Directory>' >> /etc/apache2/sites-available/000-default.conf

# Copiar código fuente desde la subcarpeta api_tienda
WORKDIR /var/www/html
COPY api_tienda/ .

# Instalar dependencias de Composer (sin dev)
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Crear la base de datos SQLite
RUN touch database/database.sqlite

# Crear .env de producción directamente (sin depender de archivo externo)
RUN echo "APP_NAME=InventMax" > .env \
    && echo "APP_ENV=production" >> .env \
    && echo "APP_KEY=" >> .env \
    && echo "APP_DEBUG=false" >> .env \
    && echo "APP_URL=https://api-tienda.onrender.com" >> .env \
    && echo "DB_CONNECTION=sqlite" >> .env \
    && echo "DB_DATABASE=/var/www/html/database/database.sqlite" >> .env \
    && echo "SESSION_DRIVER=file" >> .env \
    && echo "CACHE_STORE=file" >> .env \
    && echo "QUEUE_CONNECTION=sync" >> .env \
    && echo "LOG_CHANNEL=stack" >> .env \
    && echo "LOG_LEVEL=error" >> .env \
    && echo "FILESYSTEM_DISK=local" >> .env \
    && echo "MAIL_MAILER=log" >> .env

# Permisos correctos para Laravel
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html/storage \
    && chmod -R 755 /var/www/html/bootstrap/cache \
    && chmod 664 /var/www/html/database/database.sqlite

# Generar app key, migrar y seedear
RUN php artisan key:generate --force \
    && php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache \
    && php artisan migrate --force \
    && php artisan db:seed --force

# Exponer puerto 80 (Apache)
EXPOSE 80

# Iniciar Apache
CMD ["apache2-foreground"]
