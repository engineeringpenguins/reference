# Monitoring Services

## Introduction  

RMM (Remote Monitoring and Management) is crucial to be proactive about your infrastructure. This article will cover some options for monitoring solutions and how to install them.

## Applications

If you don’t already have a directory for docker images create one now:  
`sudo mkdir /docker`  

### Uptime Kuma

![Uptime Kuma Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/kumaDash.png)  

Uptime Kuma is one of the top monitoring solutions in the open source community. It provides a simple, elegant, and scalable interface to monitor endpoints. Native sensor types include: Ping, HTTP APIs, DNS resolvers, Databases, Docker contaniers (if you have socket exposed), Steam Game Servers. You can also build custom status pages hosted on a subdomain of the Uptime Kuma server, allowing public visability into your chosen metrics. Uptime Kuma does come with built in MFA, Proxy Support, and integrations with a handful of notification services like Slack, Email, Pushover, etc.  

Install Uptime Kuma  

1.Create the folders for persistent volumes  
a.`sudo mkdir -p /docker/UptimeKuma/app/data`  
2.Navigate to the root directory for the container  
a.`cd /docker/UptimeKuma`  
3.Create the docker compose file  
a.`sudo nano docker-compose.yaml`  
4.Paste the docker compose code below and save the document  

### Uptime KumaDocker Compose  

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

Prepare for container install  
1.Create the folders for persistent volumes  
a.`sudo mkdir -p /docker/netalertx/app/config /docker/netalertx/app/db /docker/netalertx/app/front/log`  
2.Navigate to the root directory for the container  
a.`cd /docker/netalertx`  
3.Create the docker compose file  
a.`sudo nano docker-compose.yaml`  
4.Paste the docker compose code below and save the document  

### NetAlertXDocker Compose  

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

Configuring the application  
1.Run the docker container  
a.`sudo docker compose up -d`  
2.Navigate to the ip of the website on port 20211  
3.On the left hand side open the settings menu  
4.Click on Device Scanners  
5.Set these however makes sense for your usecase  

![NetAlertX Web](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/NetAlertXGUI.png)  

## NetData

![NetData Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/netdataDash.png)

NetData touts itself as the no.1 solution due to being 50%-90% more efficient than its competitors and being designed for use 'out of the box' with minimal configuration. The product is extremely feature rich and customizeable with minimal effort, its very impressive that it runs ML against every metric in realtime and outputs a graph of anomolies.  

Install NetData  

1.Create the folders for persistent volumes  
a.`sudo mkdir -p /docker/NetData/etc`  
2.Navigate to the root directory for the container  
a.`cd /docker/NetData`  
3.Create the docker compose file  
a.`sudo nano docker-compose.yaml`  
4.Paste the docker compose code below and save the document  

### NetData Docker Compose  

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

1.Go to the NetData web interface on http port 19999  
2.Add integrations by clicking the green 'integrations' button in the upper right  
3.NetData built defaults for the server but from here you can setup other endpoints  

![NetData Sources](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/monitor/netdataMon.png)

# Work In Progress

## Grafana + Prometheus

### Grafana + Prometheus Docker Compose

## Loki

### Loki Docker Compose

## Ichinga2

### Ichinga2 Docker Compose

## Zabbix

## Nagios

## PRTG

## Munin

## Cacti
