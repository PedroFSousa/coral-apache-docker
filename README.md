# Apache Docker

This repository contains the source code for building an Apache Docker image with a custom or Let's Encrypt SSL certificate and a reverse proxy server for [OBiBa applications](https://www.obiba.org/).  

The image is based on [apache-with-certificate](https://gitlab.inesctec.pt/csig-all/apache-with-certificate) and then modified to proxy requests to the Agate, Opal, Mica and Mica Drupal applications:

- `/auth` &emsp; -> &emsp; [https://agate:8444]()  
- `/repo` &emsp; -> &emsp; [https://opal:8443]()  
- `/pub` &emsp; &ensp; -> &emsp; [https://mica:8445]()  
- `/cat` &emsp; &ensp; -> &emsp; [http://mica-drupal:80]()

## Building
After every commit, GitLab CI automatically builds the image and pushes it into INESC TEC's Docker Registry. The version tag of the image is determined from a string matching `##x.x.x##` on the commit message

Alternatively, an image can be built locally by running:
`docker build -t apache-docker .` 

## Setting Up
**Environment Variables**  
The following environment variables are required:

* `DOMAIN` is the domain (or comma-separated list of domains) to be used to access the server;
* `WEBMASTER_MAIL` is the server administrator's e-mail address;
* `CERT_TYPE` is the type of certificate to be used. More information in the `SSL Certificates` section;
* `SERVER_SYSTEM_NAME` specifies a name for the system where the image will be used. This is used to hide the server name;
* `SERVER_SYSTEM_VERSION` specifies a version for the system where the image will be used. This is used to hide the server name.
* `ENVIRONMENT` is only required if `CERT_TYPE` is set to `letsencrypt`. It specifies the environment for Let's Encrypt certificates. The default value is `dev`, which will use Let's Encrypt staging settings. If you want to deploy the image with a production-ready SSL certificate, set the variable to `prod`. More information in the `SSL Certificates` section;

**Volumes**  
Volumes can be mounted to different parts of the container:
* `/etc/apache2/conf-enabled/custom.conf` where you can place any Apache config that you may want to add;
* `/etc/ssl/certs` where you can place you custom certificates;
* `/etc/letsencrypt/` where Let's Encrypt certificates are stored (this is required to persist Let's Encrypt certificates between executions);
* `/var/www/html` where you can place all the web files that you want to serve;
* `/var/log` where you can access Apache, certbot, fail2ban and other logs.

**SSL Certificates**  
To set the type of certificate to use, set the `CERT_TYPE` environment variable with one of the following values:

* `dev` will generate a self-signed certificate for a development instance of the server. This is ***NOT*** recommended for production;
* `custom` will use a user provided certificate. The certificate files should be mounted to the certificates volume `/etc/ssl/certs`. The required files are (1) a public certificate for the server named `fullchain.pem` and (2) a private key for the server named `privkey.pem`.
* `letsencrypt` will use a certificate automatically generated from [Let's Encrypt](https://letsencrypt.org/).  
By default, `ENVIRONMENT` is set to `dev` to issue certificates from Let's Encrypt staging servers. These certificates are not valid and are meant to be used for testing purposes only. To issue a valid certificate, the `ENVIRONMENT` variable should be set to `prod`.

## Running

Create a `docker-compose.yml` file (example bellow) and run `docker-compose up`.
```yml
version: '3.2'

services:
  apache:
    image: docker-registry.inesctec.pt/coral/coral-docker-images/coral-apache-docker:1.3.0
    container_name: apache
    ports:
    - "80:80"
    - "443:443"
    environment:
    - DOMAIN=<INSERT_VALUE_HERE>
    - WEBMASTER_MAIL=<INSERT_VALUE_HERE>
    - CERT_TYPE=<INSERT_VALUE_HERE>
    - SERVER_SYSTEM_NAME=<INSERT_VALUE_HERE>
    - SERVER_SYSTEM_VERSION=<INSERT_VALUE_HERE>
    - ENVIRONMENT=<INSERT_VALUE_HERE>
    volumes:
    - <PATH_TO_CONF_FILE>/custom.conf:/etc/apache2/conf-available/custom.conf
    - <PATH_TO_CERTS>/fullchain.pem:/etc/ssl/certs/fullchain.pem # remove if not using custom CERT_TYPE
    - <PATH_TO_CERTS>/privkey.pem:/etc/ssl/certs/privkey.pem # remove if not using custom CERT_TYPE
    - <PATH_TO_HTML>:/var/www/html
    - apache-le-certs:/etc/letsencrypt/ # remove if not using letsencrypt CERT_TYPE
    - apache-logs:/var/log/

volumes:
  apache-le-certs:
  apache-logs:
```

<br>

---
master: [![pipeline status](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-apache-docker/badges/master/pipeline.svg)](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-apache-docker/commits/master)  
dev: &emsp;&ensp;[![pipeline status](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-apache-docker/badges/dev/pipeline.svg)](https://gitlab.inesctec.pt/coral/coral-docker-images/coral-apache-docker/commits/dev)
