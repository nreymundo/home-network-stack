# Unmaintained!

I haven't used this in about a year as I've moved from a RaspberryPi to a DYI computer running Unraid and an SFF build with Proxmox. 
Keeping this around as I still reference it every now and then but will likely not update it again in the near future. 

# Description

This repository contains the main  docker-compose configuration and helper files I use to manage my current home network. I used [htpcBeginner's](https://github.com/htpcBeginner) guides and their [docker-traefik](https://github.com/htpcBeginner/docker-traefik) setup as a great starting point (and stole a bunch of stuff) then tweaked it to my own needs and experimented with other stuff. 

As of writing this (and assuming I haven't updated it and forgot to update this readme) the entire thing runs on a RaspberryPi 4 4GB which honestly it's starting to show its limits, specially when it comes to I/O operations. It works and is totally usable but don't think I'd be adding much more to this until I either split the load between other rPis or, more likely, move everything to a more powerful server. 


__NOTE:__ I tried to use multi-arch containers wherever possible but for some apps I had to specify ARM images. If planning to use this in an x86/x64 system you'd have to replace those images with their suitable alternatives. 

<br>

# Deployment

## Requirements
* Docker. 
* Docker-Compose. 
* Access Control Lists (ACL) - Optional. To avoid permission issues with the containers and the host system. 

## Steps
```
mkdir ~/docker
sudo setfacl -Rdm g:docker:rwx ~/docker
sudo chmod -R 775 ~/docker
```

Creates a folder named docker under your home dir and sets it so that any new file/folder created inside are accessible for the `docker` usergroup, your own user and of course root. 
<br>
I know that permission set is way too open but to be perfectly honest I'd take the hit to security if it means I don't have to deal with permission issues for the containers or suddenly ending with root-only files. 

```
mkdir -p traefik2/acme
touch traefik2/acme/acme.json
chmod 600 traefik2/acme/acme.json
touch traefik2/traefik.log
```

Sets up the `traefik` folder structure with placeholders for the ACME challenge for the SSL certs and for the log file to then be able to use `fail2Ban`. 

```
docker network create --gateway 192.168.90.1 --subnet 192.168.90.0/24 proxy
docker network create --gateway 192.168.91.1 --subnet 192.168.91.0/24 --internal socket_proxy
docker network create --gateway 192.168.92.1 --subnet 192.168.92.0/24 apps
```

Creates 3 networks. 
* `proxy` is where I put all the apps that I want to be able to access from the web. 
* `socket_proxy` is for those containers that need to interact with the Docker Daemon. Its not accessible outside the docker context. 
* `apps` is the catch-all network in which I put everything (minus the `socket-proxy`) container. I use this so I can set up communication between containers by using Docker's internal DNS and so that I only need to explicitly expose ports for those containers that need to be accessed by other systems in my LAN. 

```
cd ~/docker
cp env.placeholder .env
```

Then you need to manually fill in all the environment variables that will be used by docker when creating the containers. 

```
docker-compose up -d
```

## Getting Authelia to Work
This is a great step-by-step guide on how to setup Authelia, create the initial user and configure Traefik to use it as the authentication system. It also has some tips to improve performance and how to make it scale up if needed. 

[Link to the guide in smarthomebeginner.com](https://www.smarthomebeginner.com/docker-authelia-tutorial/)

## Extra Configuration
* You need a domain obviously. 
* Need to set up forwarding from your router so that ports `80` and `443` are sent to the host system where all this is running. 
* Both `home-assistant` and `room-assistant` require their network access to be set to `host` to be able to discover new integrations in the network or access the bluetooth controller for room tracking. Due to this the `home-assistant` instance needs to be access directly through `http:<host-ip>:8123` and not using Docker's DNS. 

<br>

# Applications and Services

### Ingress
* Traefik2: Handles load balancing, incoming traffic, SSL certificate generation and renewal. 
* Authelia: Provides Single-Sign-On as well as access control and restriction capabilities. 

### Dashboards
* Heimdall: Application dashboard and launcher. 
* Glances: Host system information (think `htop` but browser-based). 
* Dozzle: Real time log viewer for all the docker containers. 
* Grafana: Metrics visualization and analytics. 
* Portainer: Web-based docker management tool. 

### Home Automation
* Home Assistant: Home automation hub with integrations for many different services, brands and appliances. 
* Room Assistant: Room-based presence tracking using Raspberry Pis and Bluetooth/BLE devices. 
* Node-red: Flow-based development tool used to automate IoT devices. 
* Mosquitto: MQTT broker for IoT integration. 

### Data Storage
* Redis: Key-Value storage for caching purposes. 
* MariaDB: MySQL fork. 
* InfluxDB: Time series DB used to store metrics from many different systems. 
* Prometheus: Pretty much the same as InfluxDB but different. 

### Utilities
* Socket-proxy: Allows more granular control over services wanting to access the Docker socket. 
* Watchtower: Automatically updates running docker containers. 
* OpenSpeedTest: Lets you do offline speedtest over your local network or through the internet if exposed. I mainly use it for troubleshooting local network connectivity.   
* Pi-hole: Network wide adblocking and other useful tools when used as DNS. 
