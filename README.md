[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-prestashop/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-prestashop/tree/master)

# What is PrestaShop?

PrestaShop is a popular open source e-commerce solution. Professional tools are easily accessible to increase online sales including instant guest checkout, abandoned cart reminders and automated Email marketing.

http://www.prestashop.com

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-prestashop/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Supported tags and respective `Dockerfile` links

> NOTE: Debian 8 images have been deprecated in favor of Debian 9 images. Bitnami will not longer publish new Docker images based on Debian 8.


* [`1.7-ol-7`, `1.7.4-2-ol-7-r5` (1.7/ol-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-prestashop/blob/1.7.4-2-ol-7-r5/1.7/ol-7/Dockerfile)
* [`1.7-debian-9`, `1.7.4-2-debian-9-r4`, `1.7`, `1.7.4-2`, `1.7.4-2-r4`, `latest` (1.7/Dockerfile)](https://github.com/bitnami/bitnami-docker-prestashop/blob/1.7.4-2-debian-9-r4/1.7/Dockerfile)

Subscribe to project updates by watching the [bitnami/prestashop GitHub repo](https://github.com/bitnami/bitnami-docker-prestashop).

# Prerequisites

To run this application you need [Docker Engine](https://www.docker.com/products/docker-engine) >= `1.10.0`. [Docker Compose](https://www.docker.com/products/docker-compose) is recommended with a version `1.6.0` or later.

## Run PrestaShop with a Database Container

Running PrestaShop with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run PrestaShop. You can use the following docker compose template:

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_prestashop
      - MARIADB_DATABASE=bitnami_prestashop
    volumes:
      - 'mariadb_data:/bitnami'
  prestashop:
    image: 'bitnami/prestashop:latest'
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - PRESTASHOP_DATABASE_USER=bn_prestashop
      - PRESTASHOP_DATABASE_NAME=bitnami_prestashop
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'prestashop_data:/bitnami'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  prestashop_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create prestashop-tier
  ```

2. Create a volume for MariaDB persistence and create a MariaDB container

  ```bash
  $ docker volume create --name mariadb_data
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_prestashop \
    -e MARIADB_DATABASE=bitnami_prestashop \
    --net prestashop-tier \
    --volume mariadb_data:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to PrestaShop to resolve the host

3. Create volumes for Prestashop persistence and launch the container

  ```bash
  $ docker volume create --name prestashop_data
  $ docker run -d --name prestashop -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e PRESTASHOP_DATABASE_USER=bn_prestashop \
    -e PRESTASHOP_DATABASE_NAME=bitnami_prestashop \
    --net prestashop-tier \
    --volume prestashop_data:/bitnami \
    bitnami/prestashop:latest
  ```

Then you can access your application at <http://your-ip/>

  *Note:* If you want to access your application from a public IP or hostname you need to configure PrestaShop for it. You can handle it adjusting the configuration of the instance by setting the environment variable "PRESTASHOP_HOST" to your public IP or hostname.

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define docker volumes namely `mariadb_data` and `prestashop_data`. The PrestaShop application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_prestashop
      - MARIADB_DATABASE=bitnami_prestashop
    volumes:
      - '/path/to/mariadb-persistence:/bitnami'
  prestashop:
    image: 'bitnami/prestashop:latest'
    environment:
      - PRESTASHOP_DATABASE_USER=bn_prestashop
      - PRESTASHOP_DATABASE_NAME=bitnami_prestashop
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/prestashop-persistence:/bitnami'
   depends_on:
      - mariadb
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create prestashop-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_prestashop \
    -e MARIADB_DATABASE=bitnami_prestashop \
    --network prestashop-tier \
    --volume /path/to/mariadb-persistence:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to PrestaShop to resolve the host

3. Run the PrestaShop container:

  ```bash
  $ docker run -d --name prestashop -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e PRESTASHOP_DATABASE_USER=bn_prestashop \
    -e PRESTASHOP_DATABASE_NAME=bitnami_prestashop \
    --network prestashop-tier \
    --volume /path/to/prestashop-persistence:/bitnami \
    bitnami/prestashop:latest
  ```

# Upgrading PrestaShop

Bitnami provides up-to-date versions of MariaDB and PrestaShop, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the PrestaShop container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/prestashop:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop prestashop`
 * For manual execution: `$ docker stop prestashop`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/prestashop-persistence /path/to/prestashop-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v prestashop`
 * For manual execution: `$ docker rm -v prestashop`

5. Run the new image

 * For docker-compose: `$ docker-compose up prestashop`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name prestashop bitnami/prestashop:latest`

# Configuration

## Environment variables

When you start the PrestaShop image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line.

##### User and Site configuration

 - `APACHE_HTTP_PORT_NUMBER`: Port used by Apache for HTTP. Default: **80**
 - `APACHE_HTTPS_PORT_NUMBER`: Port used by Apache for HTTPS. Default: **443**
 - `PRESTASHOP_FIRST_NAME`: PrestaShop application User's First Name. Default: **Bitnami**
 - `PRESTASHOP_LAST_NAME`: PrestaShop application User's Last Name. Default: **User**
 - `PRESTASHOP_PASSWORD`: PrestaShop application password. Default: **bitnami**
 - `PRESTASHOP_EMAIL`: PrestaShop application email. Default: **user@example.com**
 - `PRESTASHOP_HOST`: PrestaShop Host Server.
 - `PRESTASHOP_COOKIE_CHECK_IP`: Whether to check the cookie's IP address or not. Default: **yes**. See (Troubleshooting)[#Troubleshooting] section.

##### Use an existing database

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `PRESTASHOP_DATABASE_NAME`: Database name that PrestaShop will use to connect with the database. Default: **bitnami_prestashop**
- `PRESTASHOP_DATABASE_USER`: Database user that PrestaShop will use to connect with the database. Default: **bn_prestashop**
- `PRESTASHOP_DATABASE_PASSWORD`: Database password that PrestaShop will use to connect with the database. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### Create a database for PrestaShop using mysql-client

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `MARIADB_ROOT_USER`: Database admin user. Default: **root**
- `MARIADB_ROOT_PASSWORD`: Database password for the `MARIADB_ROOT_USER` user. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_NAME`: New database to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_USER`: New database user to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_PASSWORD`: Database password for the `MYSQL_CLIENT_CREATE_DATABASE_USER` user. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
prestashop:
  image: bitnami/prestashop:latest
  ports:
    - 80:80
    - 443:443
  environment:
    - PRESTASHOP_HOST=your_host
  volumes:
    - prestashop_data:/bitnami
```

 * For manual execution add a `-e` option with each variable and value:

```bash
$ docker run -d --name prestashop -p 80:80 -p 443:443 \
  --network prestashop-tier \
  --e PRESTASHOP_PASSWORD=my_password \
  --volume /path/to/prestashop-persistence:/bitnami \
  bitnami/prestashop:latest
```

## SMTP Configuration

To configure PrestaShop to send email using SMTP you can set the following environment variables:

- `SMTP_HOST`: SMTP host.
- `SMTP_PORT`: SMTP port.
- `SMTP_PROTOCOL`: SMTP protocol [ssl, tls, ""].
- `SMTP_USER`: SMTP account user.
- `SMTP_PASSWORD`: SMTP account password.

This would be an example of SMTP configuration using a GMail account:

* docker-compose:

```yaml
prestashop:
  image: bitnami/prestashop:latest
  ports:
    - 80:80
    - 443:443
  environment:
    - MARIADB_HOST=mariadb
    - MARIADB_PORT_NUMBER=3306
    - PRESTASHOP_DATABASE_USER=bn_prestashop
    - PRESTASHOP_DATABASE_NAME=bitnami_prestashop
    - SMTP_HOST=smtp.gmail.com
    - SMTP_PORT=587
    - SMTP_PROTOCOL=tls
    - SMTP_USER=your_email@gmail.com
    - SMTP_PASSWORD=your_password
```

* For manual execution:

```bash
$ docker run -d --name prestashop -p 80:80 -p 443:443 \
  -e MARIADB_HOST=mariadb \
  -e MARIADB_PORT_NUMBER=3306 \
  -e PRESTASHOP_DATABASE_USER=bn_prestashop \
  -e PRESTASHOP_DATABASE_NAME=bitnami_prestashop \
  -e SMTP_HOST=smtp.gmail.com \
  -e SMTP_PORT=587 \
  -e SMTP_PROTOCOL=tls \
  -e SMTP_USER=your_email@gmail.com \
  -e SMTP_PASSWORD=your_password \
  --network prestashop-tier \
  --volume /path/to/prestashop-persistence:/bitnami \
  bitnami/prestashop:latest
```

# Troubleshooting

* If you are automatically logged out from the administration panel, you can try deploying PrestaShop with the environment variable `PRESTASHOP_COOKIE_CHECK_IP=no`

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-prestashop/issues), or submit a
[pull request](https://github.com/bitnami/bitnami-docker-prestashop/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-prestashop/issues). For us  to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`$ docker version`)
- Output of `$ docker info`
- Version of this container (`$ echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright 2016-2018 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
