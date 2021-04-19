---
title: Nextcloud
description: Out with it Snoogle ! 
published: 1
date: 2021-04-19T05:21:41.483Z
tags: self-hosting, snoogle
editor: markdown
dateCreated: 2021-04-19T05:13:17.752Z
---


Now hosting nextcloud is a pretty vanilla setup , all you gotta do is get yourself a compose file and do a docker-compose up and you are pretty much good to go 

# Docker-compose file
```yml
version: "2.1"
services:
  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - /mnt/hdd/config:/config
      - /mnt/hdd/data:/data
    ports:
      - 993:993
      - 587:587
      - 9443:443
      - 8080:80
    restart: unless-stopped
    
    # PostgreSQL
  postgres: 
    container_name: postgres
    environment: 
      - POSTGRES_PASSWORD=PleaseChangeMeandEnterAGoodPasswordHere
      - POSTGRES_USER=nextcloud
    image: postgres:latest
    restart: always
    volumes: 
      - "/mnt/hdd/postgres/init:/docker-entrypoint-initdb.d/"
      - "/mnt/hdd/postgres/data:/var/lib/postgresql/data"
      - "/etc/localtime:/etc/localtime:ro"
    # Redis
  redis:
    container_name: redis
    image: redis:latest
    restart: always

```
> This setup assumes that your disk is mounted at /mnt/hdd which you want to use for your data storage 
{.is-info}

* Now just do a docker-compose up -d 
* While installation just select postgres as a backend with the following details






# Nginx config 
Here is the nginx config in case you want to make this available publically 
```apacheconfig
server {
    listen 80;
    server_name <domain>;
    return 301 https://$host$request_uri;  # redirect http to https
}

server {
    listen 443 ssl http2;
    server_name <domain>;

    ssl_certificate     /etc/letsencrypt/live/<domain>/fullchain.pem;  
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

    port_in_redirect off;
    proxy_buffering off;
    proxy_http_version 1.1;     # Properly proxy websocket connections
    proxy_read_timeout 300s;    # terminate websockets afer 5min of inactivity
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-Protocol $scheme;


    location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-Proto https;
            proxy_pass  http://<ip>:<port>    # Port that ghost port is listening on
    }
}


```





















