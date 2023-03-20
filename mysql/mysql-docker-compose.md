## MySQL DockerCompose

`docker-compose.yaml`

````yaml
version: '3.8'
services:
  mysql:
    image: 'mysql/mysql-server:8.0'
    ports:
      - '${FORWARD_DB_PORT:-3306}:3306'
    environment:
      MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
      MYSQL_ROOT_HOST: '%'
      MYSQL_DATABASE: '${DB_DATABASE}'
      MYSQL_USER: '${DB_USERNAME}'
      MYSQL_PASSWORD: '${DB_PASSWORD}'
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
    volumes:
      - 'vol-mysql:/var/lib/mysql'
      - './.docker/mysql/create-testing-database.sh:/docker-entrypoint-initdb.d/10-create-testing-database.sh'
    networks:
      - sail
networks:
  sail:
    driver: bridge
volumes:
  sail-mysql:
    driver: local
  sail-redis:
    driver: local
````

`.env`

````.dotenv
DB_DATABASE=sail_database
DB_USERNAME=sail
DB_PASSWORD=password
````
`.docker/mysql/create-testing-database.sh`

````.shell
#!/usr/bin/env bash

mysql --user=root --password="$MYSQL_ROOT_PASSWORD" <<-EOSQL
    CREATE DATABASE IF NOT EXISTS testing;
    GRANT ALL PRIVILEGES ON \`testing%\`.* TO '$MYSQL_USER'@'%';
EOSQL
````
