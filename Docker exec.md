
docker run -d \
  --name mysql-demo \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  mysql:8.0

  


#!/bin/bash

CONTAINER="mysql-demo"
DB="testdb"
USER="root"
PASSWORD="rootpass"
TABLE="mytable"

tail -n +2 data.csv | while IFS=',' read -r col1 col2 col3 col4
do
    docker exec "$CONTAINER" mysql \
        -u"$USER" \
        -p"$PASSWORD" \
        "$DB" \
        -e "INSERT INTO $TABLE VALUES ('$col1','$col2','$col3','$col4');"
done
