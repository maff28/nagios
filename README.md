# nagios
Monitoreo con Nagios, MySQL y NGINX usando Docker
Maria Salcedo

```html
version: '3'

services:
  nagios:
    image: jasonrivers/nagios:latest
    ports:
      - "8080:80"
    volumes:
      - /opt/monitoring/nagios/etc:/opt/nagios/etc
      - /opt/monitoring/nagios/var:/opt/nagios/var
      - nagios_plugins:/usr/lib/nagios/plugins
    environment:
      - NAGIOSADMIN_USER=nagiosadmin
      - NAGIOSADMIN_PASS=nagios123
    depends_on:
      - mysql
      - nginx

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=nagios
      - MYSQL_PASSWORD=nagios123
      - MYSQL_ROOT_HOST=%
    ports:
      - "3306:3306"
    command: 
      - --default-authentication-plugin=mysql_native_password
      - --bind-address=0.0.0.0

  nginx:
    image: nginx:latest
    ports:
      - "8081:80"
    volumes:
      - ./nginx/conf:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html

volumes:
  nagios_plugins:
