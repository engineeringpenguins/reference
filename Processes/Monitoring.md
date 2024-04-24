# Monitoring Services

## Introduction  

RMM (Remote Monitoring and Management) is crucial to be proactive about your infrastructure. This article will cover some options for monitoring solutions and how to install them.

## Applications

If you don’t already have a directory for docker images create one now:  
`sudo mkdir /docker`  

### Uptime Kuma

![Uptime Kuma Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/kumaDash.png)  

Uptime Kuma is one of the top monitoring solutions in the open source community. It provides a simple, elegant, and scalable interface to monitor endpoints. Native sensor types include: Ping, HTTP APIs, DNS resolvers, Databases, Docker contaniers (if you have socket exposed), Steam Game Servers. You can also build custom status pages hosted on a subdomain of the Uptime Kuma server, allowing public visability into your chosen metrics. Uptime Kuma does come with built in MFA, Proxy Support, and integrations with a handful of notification services like Slack, Email, Pushover, etc.  

[Uptime Kuma Repo](https://github.com/louislam/uptime-kuma) - what this documentation is based off of.

Install Uptime Kuma  

1. Create the folders for persistent volumes  
   - `sudo mkdir -p /docker/UptimeKuma/app/data`  
2. Navigate to the root directory for the container  
   - `cd /docker/UptimeKuma`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yaml`  
4. Paste the docker compose code below and save the document  

### Uptime KumaDocker Compose  

```

version: '3.3'  

services:  
  uptime-kuma:  
    image: louislam/uptime-kuma:1  
    container_name: uptime-kuma  
    volumes:  
      - /docker/UptimeKuma/app/data:/app/data  
      - /var/run/docker.sock:/var/run/docker.sock
    ports:  
      - 3001:3001  
    restart: always  

```

Configuring the application  
1.Run the docker container  
a.`sudo docker compose up -d`  
2.Navigate to the ip of the website on port 3001  
3.Click 'Add New Monitor' in the upper left  
4.Select the appropriate sensor type from the dropdown  
5.Provide the display name and hostname of the endpoint  
6.Add to any organizational groups before saving  

![Uptime Kuma Sensors](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/kumaMon.png)  

### NetAlertX (Formally pi.alert)

![NetAlertX Web](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/NetAlertXDash.png)  

PiAlert was the best original monitoring solution available for the raspberry pi. It has been forked a couple times and now exists as NetAlertX. Hardware/Binary installs are not supported and are not recommended by the developer so this is a docker project. NetAlert uses arp and nmap to discover devices but it can also integrate with other devices on your network with SNMP or vendor specific integrations like Unifi or PiHole.  

[NetAlertX Repo](https://github.com/jokob-sk/NetAlertX) - what this documentation is based off of.

Prepare for container install  

1. Create the folders for persistent volumes  
   - `sudo mkdir -p /docker/netalertx/app/config /docker/netalertx/app/db /docker/netalertx/app/front/log`  
2. Navigate to the root directory for the container  
   - `cd /docker/netalertx`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yaml`  
4. Paste the docker compose code below and save the document  

### NetAlertXDocker Compose  

```

version: "3"  
services:  
  netalertx:  
    container_name: netalertx  
    image: "jokobsk/netalertx:latest"  
    network_mode: "host"  
    restart: unless-stopped  
    volumes:  
      - /docker/NetAlertX/config:/app/config  
      - /docker/NetAlertX/app/db:/app/db  
      - /docker/NetAlertX/app/front/log:/app/front/log  
    environment:  
      - TZ=America/Denver  
      - PORT=20211  

```

Configuring the application  
1.Run the docker container  
a.`sudo docker compose up -d`  
2.Navigate to the ip of the website on port 20211  
3.On the left hand side open the settings menu  
4.Click on Device Scanners  
5.Set these however makes sense for your usecase  

![NetAlertX Web](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/NetAlertXGUI.png)  

\  

## NetData

![NetData Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/netdataDash.png)

NetData touts itself as the no.1 solution due to being 50%-90% more efficient than its competitors and being designed for use 'out of the box' with minimal configuration. The product is extremely feature rich and customizeable with minimal effort, its very impressive that it runs ML against every metric in realtime and outputs a graph of anomolies.  

[NetData Repo](https://github.com/netdata/netdata) - what this documentation is based off of.

Install NetData  

1. Create the folders for persistent volumes  
   - `sudo mkdir -p /docker/NetData/etc`  
2. Navigate to the root directory for the container  
   - `cd /docker/NetData`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yaml`  
4. Paste the docker compose code below and save the document  

### NetData Docker Compose  

```

version: '3'  
services:  
  netdata:  
    image: netdata/netdata  
    container_name: netdata  
    hostname: netdata  
    pid: host  
    network_mode: host  
    restart: unless-stopped  
    cap_add:  
      - SYS_PTRACE  
      - SYS_ADMIN  
    security_opt:  
      - apparmor:unconfined  
    volumes:  
      - /docker/netdata/etc:/etc/netdata  
      - netdatalib:/var/lib/netdata  
      - netdatacache:/var/cache/netdata  
      - /etc/passwd:/host/etc/passwd:ro  
      - /etc/group:/host/etc/group:ro  
      - /etc/localtime:/etc/localtime:ro  
      - /proc:/host/proc:ro  
      - /sys:/host/sys:ro  
      - /etc/os-release:/host/etc/os-release:ro  
      - /var/log:/host/var/log:ro  
      - /var/run/docker.sock:/var/run/docker.sock:ro  

volumes:  
  netdatalib:  
  netdatacache:  

```

### NetData Conclusion

1.Go to the NetData web interface on http port 19999  
2.Add integrations by clicking the green 'integrations' button in the upper right  
3.NetData built defaults for the server but from here you can setup other endpoints  

![NetData Sources](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/netdataMon.png)

## Grafana + Prometheus

![Grafana Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/grametheusDash.png)

Grafana and Prometheus are the iconic duo of monitoring tools despite being two seperate companies/products. Prometheus provides a realtime monitoring solution with a granular query language and Grafana provides an intuitive and versitle program to make stunning dashboards and visualize data. Grafana and Prometheus were not designed to be run in docker together and should typically be installed as system applications but we have used docker thus far so lets keep going. Its much easier to just install binaries, dont even think about using this in production.  

[Grafana Repo](https://github.com/grafana/grafana)
[Prometheus Repo](https://github.com/prometheus/prometheus)
[Third party Compose](https://dzlab.github.io/monitoring/2021/12/30/monitoring-stack-docker/) - what this documentation is based off of.

Install Grafana + Prometheus  

1. Create the folders for persistent volumes  
   - `sudo mkdir -p /docker/Grametheus`  
2. Navigate to the root directory for the container  
   - `cd /docker/Grametheus`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yml`  
4. Paste the docker compose code below and save the document  
5. Continue to create files provided in the headers below using their respective code  
   - prometheus.yml, grafana_config.ini, grafana_datasources.yml  

### Grafana + Prometheus Docker Compose (docker-compose.yml)

```

version: '3'  

volumes:
  prometheus_data: {}
  grafana_data: {}

services:

  alertmanager:
    container_name: alertmanager
    hostname: alertmanager
    image: prom/alertmanager
    volumes:
      - /docker/Grametheus/alertmanager.conf:/etc/alertmanager/alertmanager.conf
    command:
      - '--config.file=/etc/alertmanager/alertmanager.conf'
    ports:
      - 9093:9093

  prometheus:
    container_name: prometheus
    hostname: prometheus
    image: prom/prometheus
    volumes:
      - /docker/Grametheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - /docker/Grametheus/alert_rules.yml:/etc/prometheus/alert_rules.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    links:
      - alertmanager:alertmanager
    ports:
      - 9090:9090
    extra_hosts:  
      - 'promdock:host-gateway'

  grafana:
    container_name: grafana
    hostname: grafana
    image: grafana/grafana
    volumes:
      - /docker/Grametheus/grafana_datasources.yml:/etc/grafana/provisioning/datasources/all.yaml
      - /docker/Grametheus/grafana_config.ini:/etc/grafana/config.ini
      - grafana_data:/var/lib/grafana
    ports:
      - 3000:3000

### prometheus.yml

global:
  scrape_interval:     15s  
  evaluation_interval: 15s  

alerting:
  alertmanagers:
    - static_configs:
      - targets: ["promdock:9093"]

rule_files:
  - /etc/prometheus/alert_rules.yml

scrape_configs:
  - job_name: 'vad-metrics'
    metrics_path: '/metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['docker:9090']

### grafana_config.ini

[paths]
provisioning = /etc/grafana/provisioning

[server]
enable_gzip = true

### grafana_datasources.yml

apiVersion: 1

datasources:
  - name: 'prometheus'
    type: 'prometheus'
    access: 'proxy'
    url: 'http://prometheus:9090'

### alertmanager.conf

global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'job']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'file-log'

receivers:
- name: 'file-log'
  webhook_configs:
  - url: 'http://localhost:5001/'

```

### Grafana + Prometheus Conclusion

This is a very base config and does not include any configuration for the alertmanager container. But should otherwise be a good starting point to build on. No need to login to prometheus or alertmanager, all work will be done in grafana.  

1. Create empty file for alert manager
   - `sudo touch alertmanager_rules.yml`
2. Navigate to the ip of the website on port 3000 for grafana
3. Login with admin/admin
4. Open the Dashboards menu on the far left navigation
5. Click the blue '+ Create Dashboard' button in the center of the screen
6. On the following screen click the '+ Add Visualization' button
7. Select the 'Prometheus' option (should be the only option)
8. Use the metric dropdown and select your desired metric
9. Run the query using the 'Run Query' button in the middle right
10. Choose what visualization you want in the upper right
11. Continue creating visualizations and save the dashboard

![Grafana Config](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/grametheusMon.png)

## Icinga2

![Icinga2 Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/icingaDash.png)

Icinga2 is the most powerful and configurable monitoring solution I have ever used. Without a PhD from icinga university I find it to be a very difficult solution to get started with but the potential is unlimited. Historically there was a lot of C++ and programming elements to the management and operation of icinga but with the advent of their 'Director' solution you are now able to do it from a web portal.  

[Icinca2 Repo](https://github.com/Icinga/icinga2)
[Third party Compose](https://github.com/lippserd/docker-compose-icinga/tree/master) - what this documentation is based off of.

Use the following docker configuration to get things set up but it is 1000% easier and a better choice to install a binary or from source, there are some unique exchanges in the docker process here that make things needlessly complex.  

1. Create the folders for persistent volumes  
   - `sudo mkdir -p /docker/icinga2`  
2. Navigate to the root directory for the container  
   - `cd /docker/icinga2`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yml`  
4. Paste the docker compose code below and save the document  
5. Continue to create files provided in the headers below using their respective code  
   - icingadb.conf, init-icinga2.sh, init-mysql.sh  

### Icinga2 Docker Compose

```
version: '3.7'

x-icinga-db-web-config:
  &icinga-db-web-config
  icingaweb.modules.icingadb.config.icingadb.resource: icingadb
  icingaweb.modules.icingadb.redis.redis1.host: icingadb-redis
  icingaweb.modules.icingadb.redis.redis1.port: 6379
  icingaweb.modules.icingadb.commandtransports.icinga2.host: icinga2
  icingaweb.modules.icingadb.commandtransports.icinga2.port: 5665
  icingaweb.modules.icingadb.commandtransports.icinga2.password: ${ICINGAWEB_ICINGA2_API_USER_PASSWORD:-icingaweb}
  icingaweb.modules.icingadb.commandtransports.icinga2.transport: api
  icingaweb.modules.icingadb.commandtransports.icinga2.username: icingaweb
  icingaweb.resources.icingadb.charset: utf8mb4
  icingaweb.resources.icingadb.db: mysql
  icingaweb.resources.icingadb.dbname: icingadb
  icingaweb.resources.icingadb.host: mysql
  icingaweb.resources.icingadb.password: ${ICINGADB_MYSQL_PASSWORD:-icingadb}
  icingaweb.resources.icingadb.type: db
  icingaweb.resources.icingadb.username: icingadb

x-icinga-director-config:
  &icinga-director-config
  icingaweb.modules.director.config.db.resource: director-mysql
  icingaweb.modules.director.kickstart.config.endpoint: icinga2
  icingaweb.modules.director.kickstart.config.host: icinga2
  icingaweb.modules.director.kickstart.config.port: 5665
  icingaweb.modules.director.kickstart.config.username: icingaweb
  icingaweb.modules.director.kickstart.config.password: ${ICINGAWEB_ICINGA2_API_USER_PASSWORD:-icingaweb}
  icingaweb.resources.director-mysql.charset: utf8mb4
  icingaweb.resources.director-mysql.db: mysql
  icingaweb.resources.director-mysql.dbname: director
  icingaweb.resources.director-mysql.host: mysql
  icingaweb.resources.director-mysql.password: ${ICINGA_DIRECTOR_MYSQL_PASSWORD:-director}
  icingaweb.resources.director-mysql.type: db
  icingaweb.resources.director-mysql.username: director

x-icinga-web-config:
  &icinga-web-config
  icingaweb.authentication.icingaweb2.backend: db
  icingaweb.authentication.icingaweb2.resource: icingaweb-mysql
  icingaweb.config.global.config_backend: db
  icingaweb.config.global.config_resource: icingaweb-mysql
  icingaweb.config.global.module_path: /usr/share/icingaweb2/modules
  icingaweb.config.logging.log: php
  icingaweb.groups.icingaweb2.backend: db
  icingaweb.groups.icingaweb2.resource: icingaweb-mysql
  icingaweb.passwords.icingaweb2.icingaadmin: icinga
  icingaweb.resources.icingaweb-mysql.charset: utf8mb4
  icingaweb.resources.icingaweb-mysql.db: mysql
  icingaweb.resources.icingaweb-mysql.dbname: icingaweb
  icingaweb.resources.icingaweb-mysql.host: mysql
  icingaweb.resources.icingaweb-mysql.password: icingaweb
  icingaweb.resources.icingaweb-mysql.type: db
  icingaweb.resources.icingaweb-mysql.username: icingaweb
  icingaweb.roles.Administrators.groups: Administrators
  icingaweb.roles.Administrators.permissions: '*'
  icingaweb.roles.Administrators.users: icingaadmin

x-icinga2-environment:
  &icinga2-environment
  ICINGA_CN: icinga2
  ICINGA_MASTER: 1

x-logging:
  &default-logging
  driver: "json-file"
  options:
    max-file: "10"
    max-size: "1M"

networks:
  default:
    name: icinga-playground

services:
  director:
    command:
      - /bin/bash
      - -ce
      - |
        echo "Testing the database connection. Container could restart."
        (echo > /dev/tcp/mysql/3306) >/dev/null 2>&1
        echo "Testing the Icinga 2 API connection. Container could restart."
        (echo > /dev/tcp/icinga2/5665) >/dev/null 2>&1
        icingacli director migration run
        (icingacli director kickstart required && icingacli director kickstart run && icingacli director config deploy) || true
        echo "Starting Icinga Director daemon."
        icingacli director daemon run
    entrypoint: []
    logging: *default-logging
    image: icinga/icingaweb2
    restart: on-failure
    volumes:
      - icingaweb:/data

  init-icinga2:
    command: [ "/config/init-icinga2.sh" ]
    environment: *icinga2-environment
    image: icinga/icinga2
    logging: *default-logging
    volumes:
      - icinga2:/data
      - /docker/icinga2/icingadb.conf:/config/icingadb.conf
      - /docker/icinga2/icingaweb-api-user.conf:/config/icingaweb-api-user.conf
      - /docker/icinga2/init-icinga2.sh:/config/init-icinga2.sh

  icinga2:
    command: [ "sh", "-c", "sleep 5 ; icinga2 daemon" ]
    depends_on:
      - icingadb-redis
      - init-icinga2
    environment: *icinga2-environment
    image: icinga/icinga2
    logging: *default-logging
    ports:
      - 5665:5665
    volumes:
      - icinga2:/data
      - /docker/icinga2/icinga2.conf.d:/custom_data/custom.conf.d

  icingadb:
    environment:
      ICINGADB_DATABASE_HOST: mysql
      ICINGADB_DATABASE_PORT: 3306
      ICINGADB_DATABASE_DATABASE: icingadb
      ICINGADB_DATABASE_USER: icingadb
      ICINGADB_DATABASE_PASSWORD: ${ICINGADB_MYSQL_PASSWORD:-icingadb}
      ICINGADB_REDIS_HOST: icingadb-redis
      ICINGADB_REDIS_PORT: 6379
    depends_on:
      - mysql
      - icingadb-redis
    image: icinga/icingadb
    logging: *default-logging

  icingadb-redis:
    image: redis
    logging: *default-logging

  icingaweb:
    depends_on:
      - mysql
    environment:
      icingaweb.enabledModules: director, icingadb, incubator
      <<: [*icinga-db-web-config, *icinga-director-config, *icinga-web-config]
    logging: *default-logging
    image: icinga/icingaweb2
    ports:
      - 8080:8080
    # Restart Icinga Web container automatically since we have to wait for the database to be ready.
    # Please note that this needs a more sophisticated solution.
    restart: on-failure
    volumes:
      - icingaweb:/data

  mysql:
    image: mariadb:10.7
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: 1
      ICINGADB_MYSQL_PASSWORD: ${ICINGADB_MYSQL_PASSWORD:-icingadb}
      ICINGAWEB_MYSQL_PASSWORD: ${ICINGAWEB_MYSQL_PASSWORD:-icingaweb}
      ICINGA_DIRECTOR_MYSQL_PASSWORD: ${ICINGA_DIRECTOR_MYSQL_PASSWORD:-director}
    logging: *default-logging
    volumes:
      - mysql:/var/lib/mysql
      - /docker/icinga2/env/mysql/:/docker-entrypoint-initdb.d/

volumes:
  icinga2:
  icingaweb:
  mysql:

### icingadb.conf

library "icingadb"

object IcingaDB "icingadb" {
    host = "icingadb-redis"
    port = 6379
}

### icingaweb-api-user.conf

object ApiUser "icingaweb" {
  password = "$ICINGAWEB_ICINGA2_API_USER_PASSWORD"
  permissions = [ "*" ]
}

### init-icinga2.sh (create inside /docker/icinga2/env/icinga2)

#!/usr/bin/env bash

set -e
set -o pipefail

if [ ! -f /data/etc/icinga2/conf.d/icingaweb-api-user.conf ]; then
  sed "s/\$ICINGAWEB_ICINGA2_API_USER_PASSWORD/${ICINGAWEB_ICINGA2_API_USER_PASSWORD:-icingaweb}/" /config/icingaweb-api-user.conf >/data/etc/icinga2/conf.d/icingaweb-api-user.conf
fi

if [ ! -f /data/etc/icinga2/features-enabled/icingadb.conf ]; then
  mkdir -p /data/etc/icinga2/features-enabled
  cat /config/icingadb.conf >/data/etc/icinga2/features-enabled/icingadb.conf
fi

### init-mysql.sh (create inside /docker/icinga2/env/mysql)

#!/bin/sh -x

create_database_and_user() {
    DB=$1
    USER=$2
    PASSWORD=$3

    mysql --user root --password=$MYSQL_ROOT_PASSWORD <<EOS
CREATE DATABASE IF NOT EXISTS ${DB};
CREATE USER IF NOT EXISTS  '${USER}'@'%' IDENTIFIED BY '${PASSWORD}';
GRANT ALL ON ${DB}.* TO '${USER}'@'%';
EOS
}

create_database_and_user director director ${ICINGA_DIRECTOR_MYSQL_PASSWORD}
create_database_and_user icingadb icingadb ${ICINGADB_MYSQL_PASSWORD}
create_database_and_user icingaweb icingaweb ${ICINGAWEB_MYSQL_PASSWORD}

```

### Icinga2 Conclusion

Again, using this method of installation makes an already difficult product even more difficult. It will take time that I am not willing to spend to get this docker build working much past this point.  

Update file permissions

1. Update file permissions on the two scripts
   - sudo chmod +x /docker/icinga2/env/mysql/init-mysql.sh
   - sudo chmod +x /docker/icinga2/init-icinga2.sh
2. sudo docker compose -p icinga-playground up -d
3. Navigate to the ip of the website on port 8080
   - The API listens on port 5665
4. Login with icingaadmin and icinga
   - API login is icingaweb and icingaweb
5. Use the new "Director" platform to provision hosts and services

> Drop everything with: sudo docker compose -p icinga-playground down

![Icinga2 Config](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/icingaMon.png)

## Zabbix

Zabbix is a bit of a spiderweb for their docker configuration so I found it much simpler just to clone the repo and work with it locally or use a third party compose solution (which is what I do below). While certainly a very common enterprise solution I believe it is falling behind some of the earlier solutions in this article.  

[Zabbix Repo](https://github.com/zabbix/zabbix-docker)
[Third party Compose](https://github.com/USBAkimbo/Random/blob/master/Docker/zabbix.yml)

### Zabbix Docker Compose

```

version: '3.3'

services:
  zabbix-db:
    container_name: zabbix-db
    image: mariadb:10.11.4
    restart: always
    volumes:
      - ${ZABBIX_DATA_PATH}/zabbix-db/mariadb:/var/lib/mysql:rw
      - ${ZABBIX_DATA_PATH}/zabbix-db/backups:/backups
    command:
      - mariadbd
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_bin
      - --default-authentication-plugin=mysql_native_password
    environment:
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    stop_grace_period: 1m

  zabbix-server:
    container_name: zabbix-server
    image: zabbix/zabbix-server-mysql:ubuntu-6.4-latest
    restart: always
    ports:
      - 10051:10051
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/alertscripts:/usr/lib/zabbix/alertscripts:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/externalscripts:/usr/lib/zabbix/externalscripts:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/dbscripts:/var/lib/zabbix/dbscripts:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/export:/var/lib/zabbix/export:rw
      - ${ZABBIX_DATA_PATH}/zabbix-server/modules:/var/lib/zabbix/modules:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/enc:/var/lib/zabbix/enc:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/ssh_keys:/var/lib/zabbix/ssh_keys:ro
      - ${ZABBIX_DATA_PATH}/zabbix-server/mibs:/var/lib/zabbix/mibs:ro
    environment:
      - MYSQL_ROOT_USER=root
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - DB_SERVER_HOST=zabbix-db
      - ZBX_STARTPINGERS=${ZBX_STARTPINGERS}
    depends_on:
      - zabbix-db
    stop_grace_period: 30s
    sysctls:
      - net.ipv4.ip_local_port_range=1024 65000
      - net.ipv4.conf.all.accept_redirects=0
      - net.ipv4.conf.all.secure_redirects=0
      - net.ipv4.conf.all.send_redirects=0

  zabbix-web:
    container_name: zabbix-web
    image: zabbix/zabbix-web-nginx-mysql:ubuntu-6.4-latest
    restart: always
    ports:
      - 8080:8080
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${ZABBIX_DATA_PATH}/zabbix-web/nginx:/etc/ssl/nginx:ro
      - ${ZABBIX_DATA_PATH}/zabbix-web/modules/:/usr/share/zabbix/modules/:ro
    environment:
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - DB_SERVER_HOST=zabbix-db
      - ZBX_SERVER_HOST=zabbix-server
      - ZBX_SERVER_NAME=Zabbix Docker
      - PHP_TZ=Europe/London
    depends_on:
      - zabbix-db
      - zabbix-server
    stop_grace_period: 10s

```

## Nagios

## PRTG

## Munin

## Cacti

## Loki
