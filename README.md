# nagios
## Implementación de Monitoreo con Nagios, MySQL y NGINX usando Docker
## 1. Creación de Docker Compose
```yaml
```version: '3'

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
```
#### 2. Archivos de Configuración de Nagios

### 2.1 nagios.cfg
Ubicación: `sudo nano /usr/local/nagios/etc/nagios.cfg`
# Archivos de configuración
cfg_file=/opt/nagios/etc/commands.cfg
cfg_file=/opt/nagios/etc/contacts.cfg
cfg_file=/opt/nagios/etc/timeperiods.cfg
cfg_file=/opt/nagios/etc/templates.cfg
cfg_file=/opt/nagios/etc/hosts.cfg
cfg_file=/opt/nagios/etc/services.cfg
cfg_file=/opt/nagios/etc/localhost.cfg


### 2.2 hosts.cfg
```cfg
# MySQL Server
define host {
    use                     linux-server
    host_name               mysql_server
    alias                   MySQL Database Server
    address                 mysql
    check_command           check-host-alive
    max_check_attempts      5
    check_period           24x7
    notification_interval   30
    notification_period     24x7
}

# NGINX Server
define host {
    use                     linux-server
    host_name               nginx_server
    alias                   NGINX Web Server
    address                 nginx
    check_command           check-host-alive
    max_check_attempts      5
    check_period           24x7
    notification_interval   30
    notification_period     24x7
}

# Define hostgroups
define hostgroup {
    hostgroup_name          web-servers
    alias                   Web Servers
    members                 nginx_server
}

define hostgroup {
    hostgroup_name          db-servers
    alias                   Database Servers
    members                 mysql_server
}
```



