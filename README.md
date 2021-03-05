#### Installing Docker In Amazon Linux 2

```sh
$ sudo yum install docker -y 
$ sudo usermod -a -G docker ec2-user
$ sudo service docker restart
$ sudo chkconfig docker on
```
#### Installing Docker-Compose

```sh
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

#### Creating Compose Directory
```sh
$ mkdir wordpress
$ cd wordpress
```

#### Creating ngix.conf
```sh
$ cd wordpress
$ vim nginx.conf

server {
  listen 80;
  listen [::]:80;
  access_log off;
  root /var/www/html;
  index index.php;
  server_name example.com;
  server_tokens off;
  location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ /index.php?$args;
  }
  # pass the PHP scripts to FastCGI server listening on wordpress:9000
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass wordpress:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
  }
}
```

#### docker-compose.yml
```sh
version: '3'
services:
  database:
    image: mysql:5.6
    container_name: database
    restart: always
    volumes:
      - db-data:/var/lib/mysql
    networks:
       - wp-net
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    image: wordpress:php7.3-fpm-alpine
    restart: always
    container_name: wordpress
    volumes:
      - wp-data:/var/www/html
    depends_on:
      - database
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    networks:
      - wp-net

  nginx:
    image: nginx:alpine
    restart: always
    container_name: nginx
    volumes:
      - wp-data:/var/www/html/
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - wp-net

networks:
  app-net:    
volumes:
  db-data:
  wp-data:

networks:
  wp-net:
```

#### Creating Service
```sh
$ docker-compose up -d
nginx is up-to-date
database is up-to-date
wordpress is up-to-date
```

```sh
$ docker-compose ps
  Name                 Command               State         Ports       
-----------------------------------------------------------------------
database    docker-entrypoint.sh mysqld      Up      3306/tcp          
nginx       /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp
wordpress   docker-entrypoint.sh php-fpm     Up      9000/tcp     
```
