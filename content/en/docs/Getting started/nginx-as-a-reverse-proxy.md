---
title: "NGINX as a Reverse Proxy"
linkTitle: "NGINX as a Reverse Proxy"
date: 2020-08-14
description: >
  Use Nginx as a reverse proxy in front of Natlas Server
---

{{% pageinfo %}}
This section provides resources for setting up nginx as a reverse proxy for your Natlas server.
{{% /pageinfo %}}

It is not really advisable to run the flask application directly on the internet (or even on your local network). The flask application is just that, an application. It doesn't account for things like SSL certificates, and modifying application logic to add in potential routes for things like Let's Encrypt should be avoided. Luckily, it's very easy to setup a reverse proxy so that all of this stuff can be handled by a proper web server and leave your application to do exactly what it's supposed to do.

We provide an example nginx configuration file, however you should familiarize yourself with how to use and secure nginx.

1. [Installing Nginx](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)
2. [Nginx Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
3. [Using Certbot with Nginx](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx)
4. [Nginx Security Controls](https://docs.nginx.com/nginx/admin-guide/security-controls/)
5. [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)

You can grab our example config to help get started:

```bash
sudo curl https://raw.githubusercontent.com/natlas/natlas/main/natlas-server/deployment/nginx > /etc/nginx/sites-available/natlas
```
