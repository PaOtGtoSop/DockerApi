FROM php:8.1-rc-apache-bullseye

ENV ACCEPT_EULA=Y

RUN a2enmod rewrite

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

RUN apt update \
    && apt install -y apt-transport-https gnupg2 libpng-dev libzip-dev nano unzip \
    && rm -rf /var/lib/apt/lists/*

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/bin/install-php-extensions

RUN curl -o /usr/local/bin/composer https://getcomposer.org/composer-stable.phar \
    && chmod +x /usr/local/bin/composer

WORKDIR /var/www


RUN apt -y install tzdata
ENV TZ="America/Mexico_City"

EXPOSE 80 443

RUN docker-php-ext-enable sodium

RUN apt-get update  && apt-get install -y gnupg2 --no-install-recommends  && apt-get install \
-y apt-transport-https --no-install-recommends     \
&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && curl https://packages.microsoft.com/config/debian/11/prod.list > /etc/apt/sources.list.d/mssql-release.list     \
&& apt-get update     && ACCEPT_EULA=Y apt-get install -y --no-install-recommends        unixodbc         unixodbc-dev         libgss3         odbcinst         msodbcsql17         locales         libxslt-dev     && rm -r /var/lib/apt/lists/*

RUN apt-get update     \ 
&& echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen     \
&& apt-get install -y --no-install-recommends libldap2-dev libwebp-dev libpng-dev libjpeg-dev         libfreetype6-dev libicu-dev libmcrypt-dev libpq-dev libxml2-dev         supervisor nano cron ssl-cert libc-client-dev libkrb5-dev         libzip-dev         git wget libonig-dev    && rm -r /var/lib/apt/lists/*


RUN apt-get update -yqq     \
&& apt-get install -y --no-install-recommends openssl libssl-dev     && sed -i 's,^\(MinProtocol[ ]*=\).*,\1'TLSv1.0',g' /etc/ssl/openssl.cnf     && sed -i 's,^\(CipherString[ ]*=\).*,\1'DEFAULT@SECLEVEL=1',g' /etc/ssl/openssl.cnf    \
&& rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/     && docker-php-ext-configure gd         --with-freetype --with-jpeg --with-webp     && PHP_OPENSSL=yes docker-php-ext-configure imap --with-kerberos --with-imap-ssl

RUN install-php-extensions gd 
#xdebug


RUN pecl install pdo_sqlsrv sqlsrv redis apcu  && docker-php-ext-enable pdo_sqlsrv sqlsrv apcu redis

RUN install-php-extensions exif       \
                           gettext    \  
                           intl       \
                           ldap       \
                           pdo_mysql  \ 
                           pdo_pgsql  \   
                           xmlrpc     \
                           pcntl      \
                           bcmath     \
                           xsl        \
                           imap       \
                           soap       \
                           zip        \
                           ctype      \
                           json



COPY sites-available/produccion.conf /etc/apache2/sites-available/produccion.conf
#COPY sites-available/appSSL.conf /etc/apache2/sites-available/appSSL.conf

#COPY sites-available/certchainfull.pem /etc/ssl/certs/certchainfull.pem
#COPY sites-available/certchainfull2022.pem /etc/ssl/certs/certchainfull2022.pem



RUN a2dissite 000-default.conf
RUN a2ensite produccion.conf

#RUN a2enmod ssl
#RUN a2dissite default-ssl.conf
#RUN a2ensite autorregulacionSSL.conf



WORKDIR /var/www
COPY sh/docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

RUN apt clean --dry-run

CMD /docker-entrypoint.sh