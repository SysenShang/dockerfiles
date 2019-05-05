### create config
```shell
mkdir -p ~/docker/php7.1/cli

docker run --name tmp-php -d wyanlord/php:7.1-cli
docker cp tmp-php:/usr/local/etc/php/php.ini ~/docker/php7.1/cli/php.ini
docker rm -f tmp-php
```

### docker-compose.yml
```shell
version: "3"
services:
  php7.1-cli:
    container_name: php7.1-cli
    image: wyanlord/php:7.1-cli
    restart: always
    stdin_open: true
    tty: true
    volumes:
      - ~/html:/var/www/html
      - ~/docker/php7.1/cli/php.ini:/usr/local/etc/php/php.ini
    working_dir: /var/www/html
    command: sh
```
