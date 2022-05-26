---
title: "Install Nginx Proxy Manager on Raspberry Pi 3B"
date: 2021-06-21T17:49:45+05:30
draft: false
image: ""
tags: [raspberry-pi,nginx]
---



Nginx Proxy Manager acts as a reverse proxy while exposing the services to the network. Using a reverse proxy is good move to improve security and performance.

A reverse proxy is a server that sits in front of your apps and forward client requests to the apps. In other words, we will have to expose one server (Reverse Proxy server) using ports 80 or 443 and we will be able to expose any number of apps that we need.

In this post, we will try to install Nginx Proxy Manager docker image running on Raspberry Pi 3B


![image](images/2021/05/NginxProxyManager.jpg)





## Pre-requisite

1. Raspbian installed on Raspberry Pi 3B
2. Docker and Portainer installed 
3. Ports 80,81 and 443 should not be in use by any other container

## Installation

1. First to start with get the docker compose code for Nginx Proxy manager from the [Nginx website](https://nginxproxymanager.com/setup/#configuration-file).

``` yaml
version: "2"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: always
    ports:
      # Public HTTP Port:
      - '80:80'
      # Public HTTPS Port:
      - '443:443'
      # Admin Web Port:
      - '81:81'
    environment:
      # These are the settings to access your db
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "changeuser"
      DB_MYSQL_PASSWORD: "changepass"
      DB_MYSQL_NAME: "npm"
      # If you would rather use Sqlite uncomment this
      # and remove all DB_MYSQL_* lines above
      # DB_SQLITE_FILE: "/data/database.sqlite"
      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - ./data/nginx-proxy-manager:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db
  db:
    image: ghcr.io/linuxserver/mariadb
    restart: unless-stopped
    environment:
      PUID: 1001
      PGID: 1001
      TZ: "Europe/London"
      MYSQL_ROOT_PASSWORD: "changeme"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "changeuser"
      MYSQL_PASSWORD: "changepass"
    volumes:
      - ./data/mariadb:/config
```

**NOTE** : If you want to install sqlite database instead of mariaDB, remove all the parameters for MYSQL and remove the comments for DB_SQLITE_FILE as shown below.

I have installed the Nginx Proxy Manager using the sqlite database as it doesn't require another image and makes our installation simple and lightweight.

``` yaml 
version: "2"
services:
  app:
    image: jc21/nginx-proxy-manager:latest
    restart: always
    ports:
      # Public HTTP Port:
      - '80:80'
      # Public HTTPS Port:
      - '443:443'
      # Admin Web Port:
      - '81:81'
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

2. Navigate to Portainer administration console. Click on Stacks

![](/images/2021/05/Click-stacks.png "Click-stack")

3. Click on '+ Add Stack' button

![](/images/2021/05/Add-Stack.png "Add-stack")

4. Copy and paste the docker compose provided in step 1 in the 'Web editor'. 

![](/images/2021/05/deploy-docker-compose.png)

5. Click on 'Deploy the Stack' under Actions when completed.
6. When the deployment is completed successfully , the Nginx Proxy Manager should be running successfully.

![](/images/2021/05/nginx-proxy-manager-running.png "nginx-proxy-running")

7. We should now be able to login to Nginx Proxy Manager by accessing the url [http://raspberry-pi-ip:81](http://raspberry-pi-ip:81)
8. The default login id is "**admin@example.com**" and password is "changeme"
9. On successful login, we will be prompted to change the password

![admin-first-login](/images/2021/05/admin-first-login.png)

    

## Reference

- [Setup Nginx Proxy on Raspberry Pi](https://nginxproxymanager.com/setup/#running-on-raspberry-pi-arm-devices)
- [Nginx Proxy manager on Raspberry Pi](https://www.jhmelo.com/nginx-proxy-manager-raspberry)




