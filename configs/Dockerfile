#<<<<<<Dockerfile for JAVA>>>>>>>>>>
FROM 192.168.1.115:5050/java-base-17

RUN mkdir /app
COPY . /app
WORKDIR /app
RUN mvn clean -P production -s /root/.m2/settings-docker.xml package

CMD java -Duser.timezone=CAT -jar /app/target/application-0.0.1-RELEASE.jar

#<<<<<<<<<<<>>>>>>>>>><<<<<Dockerfile for PHP>>>>>>>>>>>>>>>>>>
FROM 192.168.1.115:5050/php-base:8.1.0

# ENV SETUP COMPLETE

# Set working directory
WORKDIR /var/www/current/omni_channel_vb

# Copy composer.lock and composer.json
COPY composer.lock composer.json package*.json package*.lock  /var/www/current/omni_channel_vb/

COPY nginx/conf.d/omni_channel_vb.conf /etc/nginx/sites-enabled/new.conf
COPY php/local.ini /usr/local/etc/php/conf.d/local.ini

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Add user for laravel application
#RUN groupadd -g 1000 www
#RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www/current/omni_channel_vb

# Copy existing application directory permissions
COPY --chown=www:www . /var/www/current/omni_channel_vb

# Change current user to www
# USER www

COPY horizon/horizon.conf /etc/supervisor/conf.d/horizon.conf
COPY queues/laravel-worker.conf /etc/supervisor/conf.d/laravel-worker.conf
RUN touch /var/www/current/omni_channel_vb/storage/logs/horizon.log
RUN touch /var/www/current/omni_channel_vb/storage/logs/worker.log
RUN chmod ugo+rwx /var/www/current/omni_channel_vb/storage/logs/horizon.log
RUN chmod ugo+rwx /var/www/current/omni_channel_vb/storage/logs/worker.log

#RUN chmod +x /var/www/current/omni_channel_vb/entrypoint.sh
ENTRYPOINT /var/www/current/omni_channel_vb/entrypoint.sh

#Install Vite
RUN npm install -g vite

# install packages
RUN composer install
RUN composer update
RUN composer dump-autoload
RUN npm install npm@9.5.1 -g
COPY test.env /var/www/current/omni_channel_vb/.env
RUN php artisan route:cache
RUN php artisan optimize
RUN php artisan migrate --force
RUN npm install
RUN npm run build

# Expose port 8000 and start php-fpm server
EXPOSE 8000
