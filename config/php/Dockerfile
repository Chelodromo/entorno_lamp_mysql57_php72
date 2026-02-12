FROM php:7.2-apache

# --- CORRECCIÓN REPOSITORIOS (Fix Debian Buster EOL) ---
RUN echo "deb http://archive.debian.org/debian buster main" > /etc/apt/sources.list && \
    echo "deb http://archive.debian.org/debian-security buster/updates main" >> /etc/apt/sources.list

# --- INSTALACIÓN DE EXTENSIONES ---
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Habilitar mod_rewrite
RUN a2enmod rewrite

# Instalar librerías extra (GD, Intl) ignorando validez de certificados antiguos
RUN apt-get -o Acquire::Check-Valid-Until=false update \
    && apt-get install -y --no-install-recommends \
    libpng-dev \
    libicu-dev \
    && docker-php-ext-install gd intl \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Configurar Apache para permitir .htaccess
RUN echo '<Directory /var/www/html/> \n\
    Options Indexes FollowSymLinks \n\
    AllowOverride All \n\
    Require all granted \n\
</Directory>' > /etc/apache2/conf-available/symfony.conf \
    && a2enconf symfony

WORKDIR /var/www/html