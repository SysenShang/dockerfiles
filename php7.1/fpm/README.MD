### create config
```
mkdir -p ~/docker/php7.1/fpm/php-fpm.d ~/docker/php7.1/fpm/logs

docker run --name tmp-php -d wyanlord/php:7.1-fpm
docker cp tmp-php:/usr/local/etc/php/php.ini ~/docker/php7.1/fpm/php.ini
docker cp tmp-php:/usr/local/etc/php-fpm.d/docker.conf ~/docker/php7.1/fpm/php-fpm.d/docker.conf
docker cp tmp-php:/usr/local/etc/php-fpm.d/www.conf ~/docker/php7.1/fpm/php-fpm.d/www.conf
docker cp tmp-php:/usr/local/etc/php-fpm.d/zz-docker.conf ~/docker/php7.1/fpm/php-fpm.d/zz-docker.conf
docker rm -f tmp-php
```

### docker-compose.yml
```
version: "3"
services:
  php7.1-fpm:
    container_name: php7.1-fpm
    image: wyanlord/php:7.1-fpm
    restart: always
    ports:
      - 9071:9000
    volumes:
      - ~/html:/var/www/html
      - ~/docker/php7.1/fpm/php.ini:/usr/local/etc/php/php.ini
      - ~/docker/php7.1/fpm/php-fpm.d:/usr/local/etc/php-fpm.d
      - ~/docker/php7.1/fpm/logs:/var/log
    working_dir: /var/www/html
```
