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
      - /nagios/etc:/opt/nagios/etc
      - /nagios/var:/opt/nagios/var
      - /nagios/customplugins:/opt/Custom-Nagios-Plugins
    environment:
      - NAGIOSADMIN_USER=nagiosadmin
      - NAGIOSADMIN_PASS=Teledatos2026*
    depends_on:
      - mysql
      - nginx

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=nagios
      - MYSQL_PASSWORD=Teledatos2026*
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
cfg_file=/usr/local/nagios/etc/objects/commands.cfg
cfg_file=/usr/local/nagios/etc/objects/contacts.cfg
cfg_file=/usr/local/nagios/etc/objects/timeperiods.cfg
cfg_file=/usr/local/nagios/etc/objects/templates.cfg
cfg_file=/usr/local/nagios/etc/objects/hosts.cfg
cfg_file=/usr/local/nagios/etc/objects/services.cfg

Definitions for monitoring the local (Linux) host
cfg_file=/usr/local/nagios/etc/objects/localhost.cfg



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

### 2.3 services.cfg
```cfg
# MySQL Services
define service {
    use                    generic-service
    host_name              mysql_server
    service_description    MySQL Connection Status
    check_command         check_tcp!3306
    check_interval        5
    retry_interval        1
    max_check_attempts    5
}

define service {
    use                    generic-service
    host_name              mysql_server
    service_description    MySQL Database
    check_command         check_mysql!nagios!nagios123!testdb
    check_interval        5
    retry_interval        1
    max_check_attempts    5
}

# NGINX Services
define service {
    use                    generic-service
    host_name              nginx_server
    service_description    NGINX HTTP
    check_command         check_http!-p 8081
    check_interval        5
    retry_interval        1
    max_check_attempts    5
}

define service {
    use                    generic-service
    host_name              nginx_server
    service_description    NGINX Process
    check_command         check_procs!1:!nginx
    check_interval        5
    retry_interval        1
    max_check_attempts    5
}
```
## 3. Configurar MySQL
1. Ingresar al contenedor MySQL:
```bash
docker-compose exec mysql bash
mysql -u root -p
```

2. Configurar permisos:
```sql
CREATE USER 'nagios'@'%' IDENTIFIED BY 'Teledatos2026*';
GRANT ALL PRIVILEGES ON *.* TO 'nagios'@'%';
FLUSH PRIVILEGES;
```
## 4. Accededr desde web
- URL: http://IP-DEL-SERVIDOR:8080/nagios
- Usuario: nagiosadmin
- Contraseña: Teledatos2026*
## 5. Comandos 
# Ingresar a la configuración del archivo Docker
- sudo nano docker-compose.yml
# Ingresar a la configuración de los archivos de individuales
- sudo nano /usr/local/nagios/etc/objects/commands.cfg
- sudo nano /usr/local/nagios/etc/objects/services.cfg
- sudo nano /usr/local/nagios/etc/objects/hosts.cfg
# Ingresar a la configuración de nagios
- sudo nano /usr/local/nagios/etc/nagios.cfg
# Reiniciar servicios
sudo docker-compose restart
# Ver logs
sudo docker-compose logs nagios


