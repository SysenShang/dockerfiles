### create config
```shell
mkdir -p ~/docker/phpenv7.3/nginx/conf.d \
     ~/docker/phpenv7.3/supervisord.d \
     ~/docker/phpenv7.3/php/php-fpm.d \
     ~/docker/phpenv7.3/log

docker run --name tmp-phpenv -d wyanlord/nginx:openresty-php7.3
docker cp tmp-phpenv:/usr/local/openresty/nginx/conf/nginx.conf ~/docker/phpenv7.3/nginx/nginx.conf
docker cp tmp-phpenv:/etc/nginx/conf.d/default.conf ~/docker/phpenv7.3/nginx/conf.d/www.conf
docker cp tmp-phpenv:/usr/local/etc/php/php.ini-production ~/docker/phpenv7.3/php/php.ini
docker cp tmp-phpenv:/usr/local/etc/php-fpm.d/www.conf ~/docker/phpenv7.3/php/php-fpm.d/www.conf

docker rm -f tmp-phpenv
```

### docker-compose.yml
```shell
version: "3"
services:
  nginx-php:
    container_name: nginx-php
    image: wyanlord/nginx:openresty-php7.3
    restart: always
    ports:
      - 80:80
      - 443:443
      - 9000:9000
    volumes:
      - ~/html:/var/www/html
      - ~/docker/phpenv7.3/log:/var/log
      - ~/docker/phpenv7.3/supervisord.d:/etc/supervisord.d
      - ~/docker/phpenv7.3/nginx/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf
      - ~/docker/phpenv7.3/nginx/conf.d:/etc/nginx/conf.d
      - ~/docker/phpenv7.3/php/php.ini:/usr/local/etc/php/php.ini
      - ~/docker/phpenv7.3/php/php-fpm.d:/usr/local/etc/php-fpm.d
    working_dir: /var/www/html
```
