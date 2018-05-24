# My Build History on CentOS7

Per instructions at https://wodby.com/stacks/drupal/docs/local/multiple-projects/

```
[mark@centos7 Docker]$ git clone https://github.com/wodby/docker4drupal.git SummittServices
[mark@centos7 Docker]$ cd SummittServices
```

*Decimal numbered steps below are modified from the aforementioned instructions.  Roman numeral steps were added as necessary.*

#### I. Create a directory named `main` where common services like `portainer` and `mailhog` will exist.  Build a new `main/docker-compose.yml` file from `traefik.yml` and a new `main/.env` from the original `.env` file.

```
[mark@centos7 SummittServices]$ mkdir main
[mark@centos7 SummittServices]$ cp -f traefik.yml main/docker-compose.yml
[mark@centos7 SummittServices]$ cp .env main/
```

##### 1. Create two dirs where you will host two projects. Let's name them ~~site1~~ d8 and ~~site2~~ d7

```
[mark@centos7 SummittServices]$ mkdir d8
[mark@centos7 SummittServices]$ mkdir d7
```

##### 2. Copy `docker-compose.yml` and `docker-compose.override.yml` file to both dirs (~~site1~~ d8 and ~~site2~~ d7)
```
[mark@centos7 SummittServices]$ cp docker-compose*.yml d7/
[mark@centos7 SummittServices]$ cp docker-compose*.yml d8/
```
##### 3. ~~Download~~ A copy of traefik.yml file ~~(inside of docker4drupal.tar.gz archive) from the latest stable release to~~ already exists in the `main` directory as `docker-compose.yml`.

##### 4. Edit ~~`traefik.yml`~~ `main/docker-compose.yml` and remove `-dir` references while changing ~~project1-dir_default to site1_default and project2-dir_default to site2_default~~ ALL instances of 'project1' to 'd8' and 'project2' to 'd7'. `d7_default` and `d8_default` ~~Those~~ are docker network names that are created automatically from the directory name where individual `docker-compose.yml` files are located.

##### 5. Edit ~~site1~~d8's `docker-compose.yml` file. There are 3 main things that need to be done there:  

1. In `nginx` service, under labels, change `traefik.backend=nginx` to ~~traefik.backend=site1_nginx_1~~ `traefik.backend: d8_nginx_1`. This is the name of the container. You can see that under NAMES when your have the containers running by executing `docker ps`.  
2. ~~Change `traefik.frontend.rule` from `Host:php.docker.localhost` to `Host:site1.docker.localhost`~~ Change ALL `labels` from a form of `- 'traefik.backend=portainer'` to a form like: `traefik.backend: 'portainer'`.
3. Comment out all lines of `traefik` service at the bottom of the file.

##### 6. Make similar 3 changes in ~~site2~~d7's `docker-compose.yml` file.

#### VII. Copy `.env` from the main `SummittServices` directory to each of the `d8` and `d7` directories.  

#### VIII. In EACH copy of `.env` set appropriate `PROJECT_NAME` and `PROJECT_BASE_URL` variables, AND be sure to uncomment appropriate version tags depending on the project's target Drupal version.

##### 7. Run `docker-compose up -d` in `main`, ~~site1~~ `d8` and ~~site2~~ `d7` dirs to spin up containers for ~~both~~ all three projects.

##### 8. Run stand-alone `traefik` using `docker-compose -f traefik.yml up -d` in the `SummittServices` directory to spin up Traefik reverse proxy.

Note that I found it necessary to perform a `docker network prune` command in order to remove old, unused Docker networks that had hold of necessary ports.

##### 9. Visit ~~http://site1.docker.localhost~~ http://d8.docker.localhost and ~~http://site2.docker.localhost~~ http://d7.docker.localhost (as specifed in the two `.env` files) in your browser.  The first time you do so you should be prompted to setup the `default` site in each Drupal version.

#### X. Sites http://traefik.docker.localhost:8080 responds as the network *dashboard* and http://portainer.docker.localhost responds showing ALL containers.

### Final Details

`default` site names and credentials are:

    Sites:  Drupal7, Drupal8
    Database Names:  drupal
    Database Usernames and Passwords:  drupal
    Site Admin Users:  Admin
    Site Admin Email: admin@summittservices.com
    Site Admin Passwords: Drup@!7@dmin, Drup@!8@dmin

The full startup sequence of commands is now...

```
[mark@centos7 SummittServices]$ source ../docker-destroy-all.sh
[mark@centos7 SummittServices]$ docker network prune
[mark@centos7 SummittServices]$ cd main
[mark@centos7 main]$ docker-compose up -d
[mark@centos7 SummittServices]$ cd ../d8
[mark@centos7 d8]$ docker-compose up -d
[mark@centos7 SummittServices]$ cd ../d7
[mark@centos7 d8]$ docker-compose up -d
[mark@centos7 d7]$ cd ..
[mark@centos7 SummittServices]$ docker-compose -f traefik.yml up -d
```

Note: You can achieve ALL of the above semi-automatically using:  
  `source ~/Projects/Docker/docker-restart-all.sh`


# Docker-based Drupal stack

[![Build Status](https://travis-ci.org/wodby/docker4drupal.svg?branch=master)](https://travis-ci.org/wodby/docker4drupal)
[![Wodby Slack](http://slack.wodby.com/badge.svg)](http://slack.wodby.com)
[![Wodby Twitter](https://img.shields.io/twitter/follow/wodbyhq.svg?style=social&label=Follow)](https://twitter.com/wodbyhq)

## Introduction

Docker4Drupal is a set of docker images optimized for Drupal. Use `docker-compose.yml` file from the [latest stable release](https://github.com/wodby/docker4drupal/releases) to spin up local environment on Linux, Mac OS X and Windows.

Read [**Getting Started**](http://wodby.com/stacks/drupal/docs/local/quick-start).

## Stack

[wodby/drupal-nginx]: https://github.com/wodby/drupal-nginx
[wodby/php-apache]: https://github.com/wodby/php-apache
[wodby/drupal]: https://github.com/wodby/drupal
[wodby/drupal-php]: https://github.com/wodby/drupal-php
[wodby/mariadb]: https://github.com/wodby/mariadb
[wodby/postgres]: https://github.com/wodby/postgres
[wodby/redis]: https://github.com/wodby/redis
[wodby/drupal-varnish]: https://github.com/wodby/drupal-varnish
[wodby/drupal-solr]: https://github.com/wodby/drupal-solr
[wodby/elasticsearch]: https://github.com/wodby/elasticsearch
[wodby/kibana]: https://github.com/wodby/kibana
[wodby/node]: https://github.com/wodby/node
[wodby/drupal-node]: https://github.com/wodby/drupal-node
[wodby/memcached]: https://github.com/wodby/memcached
[wodby/webgrind]: https://hub.docker.com/r/wodby/webgrind
[blackfire/blackfire]: https://hub.docker.com/r/blackfire/blackfire
[wodby/rsyslog]: https://hub.docker.com/r/wodby/rsyslog
[arachnysdocker/athenapdf-service]: https://hub.docker.com/r/arachnysdocker/athenapdf-service
[mailhog/mailhog]: https://hub.docker.com/r/mailhog/mailhog
[wodby/adminer]: https://hub.docker.com/r/wodby/adminer
[phpmyadmin/phpmyadmin]: https://hub.docker.com/r/phpmyadmin/phpmyadmin
[portainer/portainer]: https://hub.docker.com/r/portainer/portainer
[_/traefik]: https://hub.docker.com/_/traefik

The Drupal stack consist of the following containers:

| Container     | Versions           | Service name    | Image                              | Enabled by default |
| ------------- | ------------------ | ------------    | ---------------------------------- | ------------------ |
| Nginx         | 1.14, 1.13         | `nginx`         | [wodby/drupal-nginx]               | ✓                  |
| Apache        | 2.4                | `apache`        | [wodby/php-apache]                 |                    |
| Drupal        | 8, 7, 6            | `php`           | [wodby/drupal]                     | ✓                  |
| PHP           | 7.1, 7.0, 5.6, 5.3 | `php`           | [wodby/drupal-php]                 |                    |
| MariaDB       | 10.2, 10.1         | `mariadb`       | [wodby/mariadb]                    | ✓                  |
| PostgreSQL    | 10.1, 9.6          | `postgres`      | [wodby/postgres]                   |                    |
| Redis         | 4.0, 3.2           | `redis`         | [wodby/redis]                      |                    |
| Varnish       | 4.1                | `varnish`       | [wodby/drupal-varnish]             |                    |
| Node          | 9, 8               | `node`          | [wodby/node]                       |                    |
| Drupal node   | 1.0                | `nodejs`        | [wodby/drupal-node]                |                    |
| Solr          | 7.x, 6.x, 5.5, 5.4 | `solr`          | [wodby/drupal-solr]                |                    |
| Elasticsearch | 6.x, 5.6, 5.5, 5.4 | `elasticsearch` | [wodby/elasticsearch]              |                    |
| Kibana        | 6.x, 5.6, 5.5, 5.4 | `kibana`        | [wodby/kibana]                     |                    |
| Memcached     | 1.4                | `memcached`     | [wodby/memcached]                  |                    |
| Webgrind      | 1.5                | `webgrind`      | [wodby/webgrind]                   |                    |
| Blackfire     | latest             | `blackfire`     | [blackfire/blackfire]              |                    |
| Rsyslog       | latest             | `rsyslog`       | [wodby/rsyslog]                    |                    |
| AthenaPDF     | 2.10.0             | `athenapdf`     | [arachnysdocker/athenapdf-service] |                    |
| Mailhog       | latest             | `mailhog`       | [mailhog/mailhog]                  | ✓                  |
| Adminer       | 4.3                | `adminer`       | [wodby/adminer]                    |                    |
| phpMyAdmin    | latest             | `pma`           | [phpmyadmin/phpmyadmin]            |                    |
| Portainer     | latest             | `portainer`     | [portainer/portainer]              | ✓                  |
| Traefik       | latest             | `traefik`       | [_/traefik]                        | ✓                  |

Supported Drupal versions: 8 / 7 / 6

## Documentation

Full documentation is available at https://wodby.com/stacks/drupal/docs/local.

## Deployment

Deploy consistent docker-based Drupal stack with orchestrations to your own server via [![Wodby](https://www.google.com/s2/favicons?domain=wodby.com) Wodby](https://cloud.wodby.com/stackhub/ada51e9b-2204-45ee-8e49-a4151912a168/detail).

## License

This project is licensed under the MIT open source license.
