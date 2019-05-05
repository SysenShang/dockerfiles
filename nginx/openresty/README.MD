### create config
```shell
mkdir -p ~/docker/nginx/conf.d ~/docker/nginx/logs

docker run --name tmp-nginx -d wyanlord/nginx:openresty
docker cp tmp-nginx:/usr/local/openresty/nginx/conf/nginx.conf ~/docker/nginx/nginx.conf
docker cp tmp-nginx:/etc/nginx/conf.d/default.conf ~/docker/nginx/conf.d/www.conf
docker rm -f tmp-nginx
```

### docker-compose.yml
```shell
version: "3"
services:
  nginx:
    container_name: nginx
    image: wyanlord/nginx:openresty
    restart: always
    ports:
      - 80:80
    volumes:
      - ~/html:/var/www/html
      - ~/docker/nginx/nginx.conf:/usr/local/openresty/nginx/conf/nginx.conf
      - ~/docker/nginx/conf.d:/etc/nginx/conf.d
      - ~/docker/nginx/logs:/usr/local/openresty/nginx/logs
    working_dir: /var/www/html
```
