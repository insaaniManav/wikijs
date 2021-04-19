---
title: Discourse
description: A truly awesome discussion forum with a terrible documentation 
published: 1
date: 2021-04-19T05:36:44.708Z
tags: self-hosting
editor: markdown
dateCreated: 2021-04-18T14:21:57.422Z
---

# Step 1: Getting and building the docker image
* ```git clone https://github.com/discourse/discourse_docker.git```

* Now the official way is to run ./discourse-setup which will ask you a bunch of questions and then run discourse for you . But the very basic requirement for that is that ports 80 and 443 should be open and thats not the case in this guid

* Now to change how the official docker image builds you need to copy samples/standalone.yml
 	to containers/app.yml
* So ```cp samples/standalone.yml containers/app.yml```
* Go into the app.yml file and change the expose statement to 
```- "3001:80   # http```

* Scroll down and change the SMTP settings
> If you do not add the SMTP settings the container will not build
{.is-warning}
* Now run the launcher script and rebuild the app container
```./launcher rebuild app```

## Sample app.yml file
```yml
## this is the all-in-one, standalone Discourse Docker container template
##
## After making changes to this file, you MUST rebuild
## /var/discourse/launcher rebuild app
##
## BE *VERY* CAREFUL WHEN EDITING!
## YAML FILES ARE SUPER SUPER SENSITIVE TO MISTAKES IN WHITESPACE OR ALIGNMENT!
## visit http://www.yamllint.com/ to validate this file as needed

templates:
  - "templates/postgres.template.yml"
  - "templates/redis.template.yml"
  - "templates/web.template.yml"
  - "templates/web.ratelimited.template.yml"
## Uncomment these two lines if you wish to add Lets Encrypt (https)
  #- "templates/web.ssl.template.yml"
  #- "templates/web.letsencrypt.ssl.template.yml"

## which TCP/IP ports should this container expose?
## If you want Discourse to share a port with another webserver like Apache or nginx,
## see https://meta.discourse.org/t/17247 for details
expose:
  - "3001:80"   # http


params:
  db_default_text_search_config: "pg_catalog.english"

  ## Set db_shared_buffers to a max of 25% of the total memory.
  ## will be set automatically by bootstrap based on detected RAM, or you can override
  #db_shared_buffers: "256MB"

  ## can improve sorting performance, but adds memory usage per-connection
  #db_work_mem: "40MB"

  ## Which Git revision should this container use? (default: tests-passed)
  #version: tests-passed

env:
  LC_ALL: en_US.UTF-8
  LANG: en_US.UTF-8
  LANGUAGE: en_US.UTF-8
  # DISCOURSE_DEFAULT_LOCALE: en

  ## How many concurrent web requests are supported? Depends on memory and CPU cores.
  ## will be set automatically by bootstrap based on detected CPUs, or you can override
  #UNICORN_WORKERS: 3

  ## TODO: The domain name this Discourse instance will respond to
  ## Required. Discourse will not work with a bare IP number.
  DISCOURSE_HOSTNAME: 'discourse.example.com'

  ## Uncomment if you want the container to be started with the same
  ## hostname (-h option) as specified above (default "$hostname-$config")
  #DOCKER_USE_HOSTNAME: true

  ## TODO: List of comma delimited emails that will be made admin and developer
  ## on initial signup example 'user1@example.com,user2@example.com'
  DISCOURSE_DEVELOPER_EMAILS: 'me@example.com,you@example.com'

  ## TODO: The SMTP mail server used to validate new accounts and send notifications
  # SMTP ADDRESS, username, and password are required
  # WARNING the char '#' in SMTP password can cause problems!
  DISCOURSE_SMTP_ADDRESS: smtp.example.com
  #DISCOURSE_SMTP_PORT: 587
  DISCOURSE_SMTP_USER_NAME: user@example.com
  DISCOURSE_SMTP_PASSWORD: pa$$word
  #DISCOURSE_SMTP_ENABLE_START_TLS: true           # (optional, default true)
  #DISCOURSE_SMTP_DOMAIN: discourse.example.com    # (required by some providers)
  #DISCOURSE_NOTIFICATION_EMAIL: noreply@discourse.example.com    # (address to send notifications from)

  ## If you added the Lets Encrypt template, uncomment below to get a free SSL certificate
  #LETSENCRYPT_ACCOUNT_EMAIL: me@example.com

  ## The http or https CDN address for this Discourse instance (configured to pull)
  ## see https://meta.discourse.org/t/14857 for details
  #DISCOURSE_CDN_URL: https://discourse-cdn.example.com

  ## The maxmind geolocation IP address key for IP address lookup
  ## see https://meta.discourse.org/t/-/137387/23 for details
  #DISCOURSE_MAXMIND_LICENSE_KEY: 1234567890123456

## The Docker container is stateless; all data is stored in /shared
volumes:
  - volume:
      host: /var/discourse/shared/standalone
      guest: /shared
  - volume:
      host: /var/discourse/shared/standalone/log/var-log
      guest: /var/log

## Plugins go here
## see https://meta.discourse.org/t/19157 for details
hooks:
  after_code:
    - exec:
        cd: $home/plugins
        cmd:
          - git clone https://github.com/discourse/docker_manager.git

## Any custom commands to run after building
run:
  - exec: echo "Beginning of custom commands"
  ## If you want to set the 'From' email address for your first registration, uncomment and change:
  ## After getting the first signup email, re-comment the line. It only needs to run once.
  #- exec: rails r "SiteSetting.notification_email='info@unconfigured.discourse.org'"
  - exec: echo "End of custom commands"
```

# Configuring Nginx

## Sample reverse proxy file
Assuming discourse is running on port 3001. Here is a sample nginx reverse proxy file
```
server {
    listen 80;
    server_name ;
    return 301 https://$host$request_uri;  # redirect http to https
}

server {
    listen 443 ssl http2;
    server_name ;

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
            proxy_pass  http://0.0.0.0:3001;
        }
}
```
Make sure to add the ssl configuration to this or else it won't work 

Now just reload nginx and you are good to go 

```sudo nginx -s reload```


  
  