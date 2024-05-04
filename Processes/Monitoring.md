# Monitoring Services

## Introduction  

RMM (Remote Monitoring and Management) is crucial to be proactive about your infrastructure. This article will cover open source monitoring solutions and how to install them with docker on Ubuntu. Writing this in the event I forget everything about monitoring solutions and want to lab something quickly. Log aggregation, alerting, and SIEM for these solutions will be covered in a future post.  

<br>

### Table of Contents

<ol>
  <li><a href="#uptime-kuma"> Uptime Kuma</a></li>
  <li><a href="#netalertx"> NetAlertX</a></li>
  <li><a href="#netdata"> NetData</a></li>
  <li><a href="#icinga2"> Icinga2</a></li>
  <li><a href="#zabbix"> Zabbix</a></li>
  <li><a href="#nagios"> Nagios</a></li>
  <li><a href="#munin"> Munin</a></li>
  <li><a href="#cacti"> Cacti</a></li>
</ol>
</details>
<br>

## Applications

If you don’t already have a directory for docker images create one now:  
`sudo mkdir /docker`  

<br><br>

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

### Uptime Kuma Docker Compose  

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

### Uptime Kuma Conclusion

Configuring the application  
1.Run the docker container  
a.`sudo docker compose up -d`  
2.Navigate to the ip of the website on port 3001  
3.Click 'Add New Monitor' in the upper left  
4.Select the appropriate sensor type from the dropdown  
5.Provide the display name and hostname of the endpoint  
6.Add to any organizational groups before saving  

<br>

![Uptime Kuma Sensors](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/kumaMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

### NetAlertX

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

1. Run the docker container  
   - `sudo docker compose up -d`  
2. Navigate to the ip of the website on port 20211  
3. On the left hand side open the settings menu  
4. Click on Device Scanners  
5. Set these however makes sense for your usecase  

<br>

![NetAlertX Web](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/NetAlertXGUI.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

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

1. Go to the NetData web interface on http port 19999  
2. Add integrations by clicking the green 'integrations' button in the upper right  
3. NetData built defaults for the server but from here you can setup other endpoints  

<br>

![NetData Sources](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/netdataMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#readme-top" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

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

```

### prometheus.yml

```

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

```

### grafana_config.ini

```

[paths]
provisioning = /etc/grafana/provisioning

[server]
enable_gzip = true

```

### grafana_datasources.yml

```

apiVersion: 1

datasources:
  - name: 'prometheus'
    type: 'prometheus'
    access: 'proxy'
    url: 'http://prometheus:9090'

```

### alertmanager.conf

```

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

<br>

![Grafana Config](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/grametheusMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

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

```

### icingadb.conf

```

library "icingadb"

object IcingaDB "icingadb" {
    host = "icingadb-redis"
    port = 6379
}

```

### icingaweb-api-user.conf

```

object ApiUser "icingaweb" {
  password = "$ICINGAWEB_ICINGA2_API_USER_PASSWORD"
  permissions = [ "*" ]
}

```

### init-icinga2.sh (create inside /docker/icinga2/env/icinga2)

```

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

```

### init-mysql.sh (create inside /docker/icinga2/env/mysql)

```

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
   - `sudo chmod +x /docker/icinga2/env/mysql/init-mysql.sh`
   - `sudo chmod +x /docker/icinga2/init-icinga2.sh`
2. Start the Container
   - `sudo docker compose -p icinga-playground up -d`
3. Navigate to the ip of the website on port 8080
   - The API listens on port 5665
4. Login with icingaadmin and icinga
   - API login is icingaweb and icingaweb
5. Use the new "Director" platform to provision hosts and services

> Drop everything with: sudo docker compose -p icinga-playground down

<br>

![Icinga2 Config](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/icingaMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

## Zabbix

![Zabbix Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/zabbixMon.png)  

Zabbix is a bit of a spiderweb for their docker configuration so I found it much simpler just to clone the repo and work with it locally or use a third party compose solution (which is what I do below). While certainly a very common enterprise solution I believe it is falling behind some of the earlier solutions in this article.  

[Zabbix Repo](https://github.com/zabbix/zabbix-docker)  
[Third party Compose](https://github.com/USBAkimbo/Random/blob/master/Docker/zabbix.yml) - what this documentation is based off of.  

1. Create the root folder  
   - `sudo mkdir -p /docker/Zabbix/`  
2. Navigate to the root directory for the container  
   - `cd /docker/Zabbix`  
3. Create the remaining folder for persistent volumes
   - `sudo mkdir -p zabbix-db/mariadb zabbix-db/backups zabbix-server/alertscripts zabbix-server/externalscripts zabbix-server/dbscripts zabbix-server/export zabbix-server/modules zabbix-server/enc  zabbix-server/ssh_keys zabbix-server/mibs zabbix-web/nginx zabbix-web/modules/`
4. Create the docker compose file  
   - `sudo nano docker-compose.yaml`  
5. Paste the docker compose code below and save the document  

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
#      - --default-authentication-plugin=mysql_native_password
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
      - PHP_TZ=America/Denver
    depends_on:
      - zabbix-db
      - zabbix-server
    stop_grace_period: 10s

```

### Zabbix Conclusion

Define Variables

1. Create an environment variable file for docker to read
   - `sudo nano .env`
2. Define the varibles used in the compose file
   - `echo -e "MYSQL_USER=zabbix\nMYSQL_PASSWORD=zabbix\nMYSQL_ROOT_PASSWORD=FiY3G39pLqvktDbDM1mr\nZABBIX_DATA_PATH=/docker/Zabbix\nZBX_STARTPINGERS=2" | sudo tee .env`  

> I cant seem to get the zabbix-server to take the enviornment variable and build the config in /etc/zabbix/zabbix_server.conf so I left MYSQL_USER and MYSQL_PASSWORD at the defaults, dont do this in production.  

Zabbix Config  

1. Navigate to the webserver on port 8080  
   - Default login is Admin/zabbix  
2. Click on the Monitoring dropdown on the far left  
3. Select the Hosts option and click "Create Host" in the upper right  
4. I just built a basic SNMP connection but Zabbix supports most methods of discovery and monitoring  

<br>

![Zabbix Config](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/zabbixMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

## Nagios

![Nagios Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/nagiosDash.png)  

Nagios was the leader in monitoring and management for many years and is still one of the most widely deployed solutions and has an extensive dev community that build plugins and UIs. Not as aesthetically appealing as some of its competitors but it makes up for it in features and flexibility. There are a significant number of add-ons, plugins, UIs, themes, integrations, etc. that really allow the Nagios product to be more than just a monitoring solution. It is not designed for docker containers and similar to ichinga2 you will get a much more stable experience running the application directly on the OS and not with docker. The original Nagios is still maintained and is presented as an enterprise grade open source solution and it still behaves and looks like the original from the early 2000's and goes by "Nagios Core". Their fancy version that markets itself as the industry leading solution is currently called "Nagios XI" and offers much more features and integrations with the base installation. We are going to look at Nagios Core due to its simplicity and availability in a third party docker solution.  

> Nagios XI has a "free" version for 7 nodes or 100 services, there is not currently a docker image for it but maybe I will create one soon.

[Nagios Repo](https://github.com/NagiosEnterprises/nagioscore)  - no native docker solution
[Third party Compose](https://github.com/JasonRivers/Docker-Nagios) - what this documentation is based off of.  

1. Create the root volume for the container 
   - `sudo mkdir -p /docker/Nagios`  
2. Navigate to the root directory for the container  
   - `cd /docker/Nagios`  
3. Create the docker compose file  
   - `sudo nano docker-compose.yml`  
4. Paste the docker compose code below and save the document  

### Nagios Docker Compose

```
version: '3'
services:
  nagios:
    image: jasonrivers/nagios:latest
    restart: always
    ports:
      - 8080:80
    volumes:
    - nagiosetc:/opt/nagios/etc
    - nagiosvar:/opt/nagios/var
    - customplugins:/opt/Custom-Nagios-Plugins
    - nagiosgraphvar:/opt/nagiosgraph/var
    - nagiosgraphetc:/opt/nagiosgraph/etc

volumes:
    nagiosetc:
    nagiosvar:
    customplugins:
    nagiosgraphvar:
    nagiosgraphetc:
```

### Nagios Configs

You don't actually configure Nagios from the web interface (with the base installation) and primarily work with local config files. Most people leverage plugins like NRPE to enhance the configuration but we are just going to do this the traditional way.

1. Open the Nagios config directory  
   - `cd /var/lib/docker/volumes/nagios_nagiosetc/objects`  
2. Create a new config file for endpoints  
   - `sudo touch hosts.cfg`  
3. Create a new config file for services  
   - `sudo touch services.cfg`  
4. Use the templates provided below to populate the config files  

### Nagios hosts.cfg

```
define hostgroup {
    hostgroup_name    linux-hostgroup
    alias             linux servers
}
define host {
    name                     linux-template
    use                      generic-host
    check_period             24x7
    check_interval           1
    retry_interval           1
    max_check_attempts       10
    check_command            check-host-alive
    notification_period      24x7
    notification_interval    30
    notification_options     d,r
    contact_groups           admins
    hostgroups               linux-hostgroup
    register                 0
}
define host {
    use                     linux-template
    host_name               myhost
    alias                   My Host
    address                 192.168.1.100
}
```

### Nagios services.cfg

```
define service{
    name                            linux-service         
    active_checks_enabled           1                       
    passive_checks_enabled          1                       
    parallelize_check               1                       
    obsess_over_service             1                       
    check_freshness                 0                       
    notifications_enabled           1                       
    event_handler_enabled           1                       
    flap_detection_enabled          1                       
    process_perf_data               1                       
    retain_status_information       1                       
    retain_nonstatus_information    1                       
    is_volatile                     0                       
    check_period                    24x7                    
    max_check_attempts              3                       
    check_interval                  10                      
    retry_interval                  2                       
    contact_groups                  admins                 
    notification_options            w,u,c,r                 
    notification_interval           60                      
    notification_period             24x7                   
      register                      0                      
    }
define service {
    use                     linux-service
    host_name               myhost
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}
```

### Nagios Conclusion

1. Start the Docker container  
   - `sudo docker compose up -d`  
2. Navigate to the webserver on port 8080  
   - Default login is nagiosadmin/nagios  
3. View the base dashboards on nagios core using the "current status" menu on the left  

![Nagios Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/nagiosMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

## Munin

![Munin Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/muninMon.png)  

Munin is a monitoring system typically deployed with a partner program (Hunin) that handles orchestration and automation. Typically I see Munin set up as an information gathering tool and the information is piped to Nagios or Grafana for additional analysis and visualization. The usecase for Munin is the power it brings with its twin (Hunin) and that it is agent based which allows for easy configuration and management. Munin does not have docker support but we will be using a third party image.  

[Munin Repo](https://github.com/munin-monitoring/munin) - no docker support  
[Third Party Repo](https://github.com/aheimsbakk/container-munin) - What this documentation is based off of

### Munin Docker Compose

```
version: "3"

services:
  munin-server:
    container_name: "munin-server"
    image: aheimsbakk/munin-alpine
    restart: unless-stopped
    ports:
      - 80:80
    environment:
      NODES: server1:10.5.11.180
      TZ: America/Denver
    volumes:
      - /docker/Munin/etc/munin/munin-conf.d:/etc/munin/munin-conf.d 
      - /docker/Munin/etc/munin/plugin-conf.d:/etc/munin/plugin-conf.d 
      - /docker/Munin/var/lib/munin:/var/lib/munin 
      - /docker/Munin/var/log/munin:/var/log/munin  
```

### Munin Monitoring Configuration

Setup Munin agent on remote endpoint

1. Install the Munin agent on a host you want to monitor
   - `sudo sudo apt install munin-node`
2. Configure the Munin agent
   - `sudo nano /etc/munin/munin-node.conf`
3. Add the ip address of your Munin controller with an "allow" statement
   - RegEx in the form of ^172\.28\.1\.1$
4. Restart Munin Agent
   - `sudo systemctl restart munin-node`

We handled setting the remote endpoint on the Munin controller in the docker compose file but if you need to add anything after the fact: `sudo nano/docker/Munin/etc/munin/munin-conf.d/nodes.conf`  

### Munin Conclusion

1. Start the Docker container  
   - `sudo docker compose up -d`  
2. Navigate to the webserver on port 80  
   - Give it a minute if no immediate response  
3. Click on the host you created (server1) to access the graphs  

![Munin Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/muninMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

---

<br>

## Cacti

![Cacti Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/cactiDash.png)  

Cacti is a very old monitoring tool that has continued to compete with Nagios for decades. It bosts a similar support and dev community to Nagios but provides much simpler configuration and management at the cost of less features (although very minimal functionality is lost). They do not natively support docker so to keep with the recurring theme in this article I will advise against using unsupported docker images in production.  

[Cacti Repo](https://github.com/Cacti/cacti) - No docker support  
[Third Party Repo](https://github.com/scline/docker-cacti/tree/master) - What this documentation is based off of  

### Cacti Docker Compose

```
version: '3.5'
services:


  cacti:
    image: "smcline06/cacti"
    container_name: cacti
    hostname: cacti
    ports:
      - "80:80"
      - "443:443"
    environment:
      - DB_NAME=cacti_master
      - DB_USER=cactiuser
      - DB_PASS=cactipassword
      - DB_HOST=db
      - DB_PORT=3306
      - DB_ROOT_PASS=rootpassword
      - INITIALIZE_DB=1
      - TZ=America/Denver
    volumes:
      - cacti-data:/cacti
      - cacti-spine:/spine
      - cacti-backups:/backups
    links:
      - db


  db:
    image: "mariadb:10.3"
    container_name: cacti_db
    hostname: db
    ports:
      - "3306:3306"
    command:
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --max_connections=200
      - --max_heap_table_size=128M
      - --max_allowed_packet=32M
      - --tmp_table_size=128M
      - --join_buffer_size=128M
      - --innodb_buffer_pool_size=1G
      - --innodb_doublewrite=ON
      - --innodb_flush_log_at_timeout=3
      - --innodb_read_io_threads=32
      - --innodb_write_io_threads=16
      - --innodb_buffer_pool_instances=9
      - --innodb_file_format=Barracuda
      - --innodb_large_prefix=1
      - --innodb_io_capacity=5000
      - --innodb_io_capacity_max=10000
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - TZ=America/Denver
    volumes:
      - cacti-db:/var/lib/mysql


volumes:
  cacti-db:
  cacti-data:
  cacti-spine:
  cacti-backups:
```

### Cacti Conclusion

1. Start the Docker container  
   - `sudo docker compose up -d`  
2. Navigate to the webserver on port 80  
   - Default login is admin/admin  
3. Set new admin password when prompted  
4. Set your theme and accept EULA on the following screen  
5. Review the pre-run checks and continue  
6. Select 'new primary server' and proceed to the next screen  
7. Accept any prechecks and script execution approval  
8. For the 'default automation network' specify a cidr range for auto detection  
9. Accept default templates and additional prechecks  
10. When prompted check the box indicating that you are ready to install and continue  
11. On the console page click on 'Create' and then 'New Device' using the menu on the right  
12. Create any devices manually and create graphs for them using the same menu  

![Cacti Example](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/cactiMon.png)  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
