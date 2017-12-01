# Docker + October CMS

[![Build Status](https://travis-ci.org/aspendigital/docker-octobercms.svg?branch=master)](https://travis-ci.org/aspendigital/docker-octobercms) [![Docker Hub Pulls](https://img.shields.io/docker/pulls/aspendigital/octobercms.svg)](https://hub.docker.com/r/aspendigital/octobercms/) [![October CMS Build 428](https://img.shields.io/badge/October%20CMS%20Build-428-red.svg)](https://github.com/octobercms/october) [![Edge Build 428](https://img.shields.io/badge/Edge%20Build-428-lightgrey.svg)](https://github.com/octobercms/october)

The docker images defined in this repository serve as a starting point for [October CMS](https://octobercms.com) projects.

Based on [official docker PHP images](https://hub.docker.com/_/php), images include dependencies required by October, Composer and install the [latest release](https://octobercms.com/changelog).

## Supported Tags

- `build.428-php7.1-apache`, `php7.1-apache`, `build.428`, `latest`: [php7.1/apache/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php7.1/apache/Dockerfile)
- `build.428-php7.1-fpm`, `php7.1-fpm`: [php7.1/fpm/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php7.1/fpm/Dockerfile)
- `build.428-php7.0-apache`, `php7.0-apache`: [php7.0/apache/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php7.0/apache/Dockerfile)
- `build.428-php7.0-fpm`, `php7.0-fpm`: [php7.0/fpm/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php7.0/fpm/Dockerfile)


### Edge Tags

- `edge-build.428-php7.1-apache`, `edge-php7.1-apache`, `edge-build.428`, `edge`: [php7.1/apache/Dockerfile.edge](https://github.com/aspendigital/docker-octobercms/blob/master/php7.1/apache/Dockerfile.edge)
- `edge-build.428-php7.1-fpm`, `edge-php7.1-fpm`: [php7.1/fpm/Dockerfile.edge](https://github.com/aspendigital/docker-octobercms/blob/master/php7.1/fpm/Dockerfile.edge)
- `edge-build.428-php7.0-apache`, `edge-php7.0-apache`: [php7.0/apache/Dockerfile.edge](https://github.com/aspendigital/docker-octobercms/blob/master/php7.0/apache/Dockerfile.edge)
- `edge-build.428-php7.0-fpm`, `edge-php7.0-fpm`: [php7.0/fpm/Dockerfile.edge](https://github.com/aspendigital/docker-octobercms/blob/master/php7.0/fpm/Dockerfile.edge)

### Legacy Tags

> October CMS build 420+ requires PHP version 7.0 or higher

- `build.419-php5.6-apache`, `php5.6-apache`: [php5.6/apache/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php5.6/apache/Dockerfile)
- `build.419-php5.6-fpm`, `php5.6-fpm`: [php5.6/fpm/Dockerfile](https://github.com/aspendigital/docker-octobercms/blob/master/php5.6/fpm/Dockerfile)


## Quick Start

To run October CMS using Docker, start a container using the latest image, mapping your local port 80 to the container's port 80:

```shell
$ docker run -p 80:80 --name october aspendigital/octobercms:latest
```

 - Visit [http://localhost](http://localhost) using your browser.
 - Login to the [backend](http://localhost/backend) with the username `admin` and password `admin`.

Running a container in the foreground will dump logs to your terminal. Hit `CTRL-C` to stop the container.

> If there is a port conflict, you will receive an error message from the Docker daemon. Try mapping to an open local port (-p 8080:80) or shut down the container or server that's on the desired port.

Run the container in detached mode using the container name `october`:

```shell
$ docker run -p 80:80 --name october -d aspendigital/octobercms:latest
$ docker stop october  # Stops the container. To restart `docker start october`
$ docker rm october  # Destroys the container
```


## Database Support

On build, an SQLite database is [created and initialized](https://github.com/aspendigital/docker-octobercms/blob/d3b288b9fe0606e32ac3d6466affd2996394bdca/Dockerfile.template#L54-L57) for the Docker image. With that database, users have immediate access to the backend for testing and developing themes and plugins. However, changes made to the built-in database will be lost once the container is stopped and removed.

When projects require a persistent SQLite database, copy or create a new database to the host which can be used as a bind mount.

```shell
# Create and provision a new SQLite database:
$ touch storage/database.sqlite
$ docker run \
  -v $(pwd)/storage/database.sqlite:/var/www/html/storage/database.sqlite \
  aspendigital/octobercms php artisan october:up

# Now run with the volume mounted to your host
$ docker run -p 80:80 --name october \
 -v $(pwd)/storage/database.sqlite:/var/www/html/storage/database.sqlite \
 aspendigital/octobercms
```

Alternatively, you can host a database remotely using mysql or postgres:


```
$ docker run -p 80:80 --name october \
  -e DB_TYPE=mysql \
  -e DB_HOST=example.rds.amazonaws.com \
  -e DB_DATABASE=example \
  -e DB_USERNAME=username \
  -e DB_PASSWORD=password \
  aspendigital/octobercms
```

Or host the database using another container via `docker-compose`:

```yml
version: '2.2'
services:
  web:
    image: aspendigital/octobercms:latest
    ports:
      - 80:80
    environment:
      - TZ=${TZ:-America/Denver}
      - DB_TYPE=mysql
      - DB_HOST=mysql
      - DB_DATABASE=octobercms
      - DB_USERNAME=october
      - DB_PASSWORD=october

  mysql:
    image: mysql:latest
    ports:
      - 3306:3306
    environment:
      - TZ=${TZ:-America/Denver}
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=octobercms
```


## Cron

You can start a cron process by setting the environment variable `ENABLE_CRON` to `true`:

```shell
$ docker run -p80:80 -e ENABLE_CRON=true aspendigital/octobercms:latest
```

Using `docker-compose`:

```yml
version: '2.2'
services:
  web:
    image: aspendigital/octobercms:latest
    ports:
      - 80:80
    environment:
      - ENABLE_CRON=true
```

Separating the cron process into it's own container for production via `docker-compose`:

```yml
version: '2.2'
services:
  web:
    image: aspendigital/octobercms:latest
    init: true
    restart: always
    ports:
      - 80:80
    environment:
      - TZ=America/Denver
    volumes:
      - ./.env:/var/www/html/.env
      - ./plugins:/var/www/html/plugins
      - ./storage/app:/var/www/html/storage/app
      - ./storage/logs:/var/www/html/storage/logs
      - ./storage/database.sqlite:/var/www/html/storage/database.sqlite
      - ./themes:/var/www/html/themes

  cron:
    image: aspendigital/octobercms:latest
    init: true
    restart: always
    command: [cron, -f]
    environment:
      - TZ=America/Denver
    volumes_from:
      - web
```

## Command Line Tasks

Run the container in the background using the container name `october` and launch an interactive shell (bash) for the container.


```shell
$ docker run -p 80:80 --name october -d aspendigital/octobercms:latest

$ docker exec -it october bash
```


## App Environment

By default, `APP_ENV` is set to `docker`.

On image build, a default `.env` is [created](https://github.com/aspendigital/docker-octobercms/blob/d3b288b9fe0606e32ac3d6466affd2996394bdca/Dockerfile.template#L52) and [config files](https://github.com/aspendigital/docker-octobercms/tree/master/config/docker) for the `docker` app environment are copied to `/var/www/html/config/docker`. Environment variables can be used to override the included default settings via [`docker run`](https://docs.docker.com/engine/reference/run/#env-environment-variables) or [`docker-compose`](https://docs.docker.com/compose/environment-variables/).

> __Note__: October CMS settings stored in a site's database override the config. Active theme, mail configuration, and other settings which are saved in the database will ultimately override configuration values.


### Environment Variables


Environment variables can be passed to both docker-compose and October CMS.

 > Database credentials and other sensitive information should not be committed to the repository. Those required settings should be outlined in __.env.example__

 > Passing environment variables via Docker can be problematic in production. A `phpinfo()` call may leak secrets by outputting environment variables.  Consider mounting a `.env` volume or copying it to the container directly.


#### Docker Entrypoint

| Variable | Default | Action |
| -------- | ------- | ------ |
| ENABLE_CRON | false | Enables a cron process |
| FWD_REMOTE_IP | false | Forwards remote IP from proxy (Apache) |

#### October CMS app environment config

List of variables used in `config/docker`

| Variable | Default |
| -------- | ------- |
| APP_DEBUG | true |
| APP_URL | http://localhost |
| APP_KEY | 0123456789ABCDEFGHIJKLMNOPQRSTUV |
| CACHE_STORE | file |
| CMS_ACTIVE_THEME | demo |
| CMS_EDGE_UPDATES | false  (true in `edge` images) |
| CMS_DISABLE_CORE_UPDATES | true |
| CMS_BACKEND_SKIN | Backend\Skins\Standard |
| CMS_LINK_POLICY | detect |
| CMS_BACKEND_FORCE_SECURE | false |
| DB_TYPE | sqlite |
| DB_SQLITE_PATH | storage/database.sqlite |
| DB_HOST | mysql* |
| DB_PORT | - |
| DB_DATABASE | - |
| DB_USERNAME | - |
| DB_PASSWORD | - |
| DB_REDIS_HOST | redis* |
| DB_REDIS_PASSWORD | null |
| DB_REDIS_PORT | 6379 |
| MAIL_DRIVER | log |
| MAIL_SMTP_HOST | - |
| MAIL_SMTP_PORT | 587 |
| MAIL_FROM_ADDRESS | no-reply@domain.tld |
| MAIL_FROM_NAME | October CMS |
| MAIL_SMTP_ENCRYPTION | tls |
| MAIL_SMTP_USERNAME | - |
| MAIL_SMTP_PASSWORD | - |
| QUEUE_DRIVER | sync |
| SESSION_DRIVER | file |
| TZ\** | UTC |

<small>\* When using a container to serve a database, set the host value to the service name defined in your docker-compose.yml</small>

<small>\** Timezone applies to both container and October CMS  config</small>

---

![October](https://raw.githubusercontent.com/aspendigital/docker-octobercms/master/aspendigital-octobercms-docker-logo.png)
