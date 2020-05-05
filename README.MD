# Docker Tutorial 

1. build our docker container
	docker-compose up 

2. go to localhost:8080
	
## How it works

### Create our database Dockerfile in ./db

```
FROM mysql:8.0
```

### Create our web Dockerfile in ./www

```
FROM php:7.2-apache

RUN docker-php-ext-install mysqli 
RUN docker-php-ext-enable mysqli

RUN docker-php-ext-install xdebug 
RUN docker-php-ext-enable xdebug
```

### Create a database environment file

```
MYSQL_USER=sys_admin
MYSQL_PASSWORD=sys_password
MYSQL_ROOT_PASSWORD=root_password
```

### Setup our network in the docker compose file

```
version: "3"
networks:
  dev-frontend:
    driver: bridge  /** bridge driver is needed **/
    ipam:
      driver: default
      config:
        - subnet: 172.10.1.0/24  
  dev-backend:
    driver: bridge /** bridge driver is needed **/
    ipam: 
      driver: default
      config:
        - subnet: 172.10.2.0/23
```

 We Create two networks. A network to serve our frontend and a network to serve out backend.


### Add the database container
now we set up the services. These services will run on the networks we created previously

```

services:
  dev-db:
    build: ./db
    command: --default-authentication-plugin=mysql_native_password
    ports:
      - 3306:3306
    networks:
      dev-backend:
        ipv4_address: 172.10.3.2
    env_file:
      - ./development.env
```
This is our mysql service
the first service is named dev-db.
the network it will run on is dev-backend
we can also include a env file

### Add the web container

```
  dev-www:
    build: ./www
    ports:
      - 8080:80
    volumes:
      - ./src:/var/www/html/
    networks:
      dev-backend:
        ipv4_address: 172.10.2.2
      dev-frontend:
        ipv4_address: 172.10.1.2
    depends_on:
      - dev-db
    env_file:
      - ./development.env
```

this is our web container that
we name it dev-www
we use port 8080
the networks we use is dev-backend and dev-front-end. Notice how our backend service is serving our db and our backend.
we add our dependent dev-db

### PHP Script to test the networking

```
<?php

$mysqli = new mysqli('dev-db', getenv('MYSQL_USER'), getenv('MYSQL_PASSWORD'), 'information_schema');

if ($mysqli->connect_error) {
    echo 'Connection Error [', $mysqli->connect_errno, ']: ', $mysqli->connect_error;
} else {
    echo 'MySQLi Connected Successfully!';
}
```

### Start the containers, networks and volumes

```
docker-compose up
```
Then go to localhost:8080

### To destroy the setup

```
docker-compose down
```

### Resources

* [Docker - Network](https://docs.docker.com/network/)
* [Docker - Environment Variables](https://docs.docker.com/compose/environment-variables/)