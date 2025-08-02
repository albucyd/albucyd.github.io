---
layout: post
permalink: /2025/08/01/mailman-docker-plesk/
title: Setting up Mailman on Ubuntu 24.04 LTS with Plesk
description: Some notes on how to set up Mailman in a docker container on Ubuntu 24.04 managed with Plesk
date: 2025-08-01 13:50:58 -0000
last_modified_at: 2025-08-01 14:06:56 -0000
publish: true
pin: false
categories:
- Hacking
tags:
- admin
- plesk
- ubuntu
- docker
---
	 	 	 	  
Its taken a longer while than I had hoped. I'm not exactly very experienced with 
Linux administration and had not done much in that area in the last couple of years.

But after neglecting my personal website for way too long, I decided it was time 
to update it. I got rid of the wordpress sites on it, and replaced one with a 
static wordpress export since I do not expect it to change much any time soon.

My own site wordpress site I converted into a Jekyll based site mostly because 
I like the fact that no dynamic language is required to serve it and because I 
find the idea of a website residing in a git repository comforting. I can revert 
changes and should something seriously break, I have an always up to date backup.

But since the virtual server I’m using to serve my websites is based on 
Ubuntu 24.04 with Plesk administration software, and mailman packages for 
Ubuntu 24.04 are not available I had to manually install mailman. I decided to 
use the [docker version by maxking](https://github.com/maxking/docker-mailman).

What follows are the steps I took to get Mailman 3 running on Ubuntu 24.04 with Plesk.

# **Configuring Plesk**

1. Install [nginx](https://nginx.org/) using the *Add 	Components* section in Plesk’s *Tools and Settings*  
2. Create a subdomain [https://lists.minimeta.de](https://lists.minimeta.de/) 	
   in Plesk and secure it with a Let’s Encrypt certificate using the 	Plesk *SSL It!* Extension  
3. Turn off Apache for that subdomain so that nginx serves 	everything for that subdomain  
4. Add the following config to Additional Nginx Directives:

```
location /static {  
  alias /opt/mailman/web/static;  
  autoindex off;  
  auth_basic "Restricted";  
  auth_basic_user_file /etc/nginx/.htpasswd;  
}

location / {  
  uwsgi_pass localhost:8080;  
  include uwsgi_params;  
  uwsgi_read_timeout 300;  
  auth_basic "Restricted";  
  auth_basic_user_file /etc/nginx/.htpasswd;  
}
```
Note: I’m protecting the subdomain with a password since this is my private 
mailing list and nobody needs to be able to see it except me. In other cases, 
the <code>auth_basic</code> lines would not be required.  
To generate the password file run  
<code>htpasswd -c /etc/nginx/.htpasswd &lt;username&gt; </code>  
as root on your server. 
You cannot use the *Protect Directory* feature in Plesk since is seems 
to override the directives listed above.

# **Configuring the docker container**

Basically, I followed the directions on the [github page](https://github.com/maxking/docker-mailman) 
for maxking’s mailman docker containers. But instead of editing the 
<code>docker-compose.yaml</code> from the page, I created a compose-overlay.yaml file and 
placed it into the same folder as <code>docker-compose.yaml</code>.

The contents of this file are as follows:
```
services:

mailman-core:

environment:
- DATABASE_URL=postgresql://mailman:<DBpassword>@database/mailmandb
- HYPERKITTY_API_KEY=<hyperkitty-secret>
- TZ=Europe/Berlin
- MTA=postfix
restart: always

networks:
- mailman

mailman-web:

environment:
- DATABASE_URL=postgresql://mailman:<DBpassword>@database/mailmandb
- HYPERKITTY_API_KEY=<hyperkitty-secret>
- TZ=Europe/Berlin
- SECRET_KEY=<django-secret-key>
- SERVE_FROM_DOMAIN=lists.minimeta.de # e.g. lists.example.org
- MAILMAN_ADMIN_USER=<admin_name> # the admin user
- MAILMAN_ADMIN_EMAIL=<admin_email> # the admin mail address
- UWSGI_STATIC_MAP=/static=/opt/mailman-web-data/static
restart: always

database:

environment:
- POSTGRES_PASSWORD=<DBpassword>
restart: always
```
Replace the variables in <code><></code> with your own...

# **Configuring mailman-web**

To properly configure mailman-web, I placed the following <code>settings_local.py</code> file into <code>/opt/mailman/web</code>
```
# locale
LANGUAGE_CODE = 'de-de'

# disable social authentication
MAILMAN_WEB_SOCIAL_AUTH = []
ALLOWED_HOSTS = ['lists.minimeta.de', 'localhost', 'mailman-web']

# change it
DEFAULT_FROM_EMAIL = '<admin_email>'
DEBUG = False # Mailman-web did not work for me with this set to True

HAYSTACK_CONNECTIONS = {
    'default': {
    'ENGINE': 'xapian_backend.XapianEngine',
    'PATH': "/opt/mailman-web-data/fulltext_index",
  },
}
```
# **Configuring Postfix**

This took me quite some time to figure out, but with the help of 
[this page](https://brakkee.org/site/2022/09/17/migrating-mailman-to-k8s/), 
I finally was able to produce a working postfix configuration.

Here are the lines I added and / or changed in <code>/etc/postfix/main.cf</code>:
```
mynetworks = 172.19.199.3, 172.19.199.1
virtual_mailbox_maps = , hash:/var/spool/postfix/plesk/vmailbox regexp:/opt/mailman/core/var/data/postfix_lmtp
transport_maps = , hash:/var/spool/postfix/plesk/transport regexp:/opt/mailman/core/var/data/postfix_lmtp
relay_domains = regexp:/opt/mailman/core/var/data/postfix_domains
```

This is slightly different from the configuration proposed 
by [maxking](https://github.com/maxking/docker-mailman#postfix), but works 
together with the Plesk setup.

# **Running it**

After all this, I started the docker container with  
<code>docker compose up -d</code>  
and restarted postfix with  
<code>systemctl restart postfix</code>

I’m writing this down so I know what I do the next time I need this and in case someone else finds this helpful.

