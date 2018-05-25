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

# Default Sites Status

At this point I have a working pair of D7 and D8 `default` sites named `Drupal7` and `Drupal8`, fronted by a `traefik` network proxy, a `portainer` management console, and an unconfirmed `mailhog` service.  All in Docker and based on Docker4Drupal.  There's currently no `https` or certs of any kind.  The Drupal sites have not been extended with any contributed or custom modules, and only the `default` databases and sites exist.

The project has been pushed to https://github.com/SummittDweller/SummittServices/tree/default_sites_no_https for safe-keeping.  

# Next Steps

1. Add self-signed certs and *https://*.
2. Extend `default` sites with necessary contrib modules.
3. Try the config/profile process at https://www.chapterthree.com/blog/installing-drupal-8-from-configuration on the `Drupal8` site.
4. Create a `wieting` Drupal 8 site using a mix of `default` code and files/data from https://wieting.TamaToledo.com.




# Default Sites with HTTPS

The discussion and configuration posted at
https://github.com/wodby/docker4drupal/issues/50#issuecomment-291749340 informs this development.  The code snippet for reconfiguration of Traefik is:

```
traefik:
    image: traefik
    restart: unless-stopped
    command: -c /dev/null --web --docker --logLevel=INFO --defaultEntryPoints='https' --entryPoints="Name:https Address::443 TLS:/certs/cert.pem,/certs/key.pem" --entryPoints="Name:http Address::80 Redirect.EntryPoint:https"
    ports:
      - '80:80'
      - '443:443'
      - '8080:8080'
    volumes:
      - ./certs:/certs/
      - /var/run/docker.sock:/var/run/docker.sock
```

Steps taken...
1. Copied `../Sites/certs` directory to `SummittServices/certs`.
2. Modified `SummittServices/traefik.yml` to include portions like above.
3. [mark@centos7 Docker]$ source docker-restart-all.sh
4. Initialize both the `Drupal7` and `Drupal8` sites as before.

__It works!__

As far as I can remember, the contents of `certs`, two files, were generated at https://www.selfsignedcertificate.com.

Note that after the *docker-restart-all.sh* the Traefik dashboard still works at http://traefik.docker.localhost:8080/dashboard/ but all others must use `https://` and they must add exceptions during the first visit.

The new site addresses are:  
http://traefik.docker.localhost:8080/dashboard/
https://portainer.docker.localhost/#/dashboard  
https://mailhog.docker.localhost/  
https://d8.docker.localhost  
https://d7.docker.localhost  

This work saved as https://github.com/SummittDweller/SummittServices/tree/default_sites_with_https.

# Extending the Drupal8 Site with Contrib Modules

Some good advice/guidance can be found at https://www.lullabot.com/articles/drupal-8-composer-best-practices.

Opened a terminal into the `d8_php` container from **Portainer**.  Then from the `wodby@php.container:/var/www/html $` prompt...

```
composer require drupal/bootstrap drupal/profile drupal/pathauto drupal/calendar drupal/views_templates
composer require drupal/google_analytics drupal/embed drupal/entity_embed drupal/taxonomy_menu drupal/blog
composer require drupal/hms_field drupal/entity_clone drupal/rules drupal/redirect
composer require drupal/media_entity drupal/media_entity_image drupal/entity_browser drupal/media_entity_flickr drupal/exif
composer require drupal/backup_migrate drupal/simplenews drupal/permissions_by_term drupal/social_media drupal/imce
composer require drupal/mailgun
composer require --dev drupal/devel drupal/examples
composer require doctrine/annotations
composer update
```

Then `nano composer.json` and I added the following two lines to the `extras` | `installer-paths` portion:

```
"web/sites/all/modules/custom/{$name}/": ["type:drupal-custom-module"],
"web/sites/all/themes/custom/{$name}/": ["type:drupal-custom-theme"],
```

Then I downloaded the current `composer.json` from the PHP container, edited it in Atom adding the parts shown below to the `repositories` key like so...
```
[mark@centos7 SummittServices]$ docker cp d8_php:/var/www/html/composer.json .  
...edit in Atom...  
[mark@centos7 SummittServices]$ docker cp ./composer.json d8_php:/var/www/html/composer.json
```

The working `composer.json` file is now held in my `~/Projects/Docker/SummittServices` project directory.

### Added Modules and Theme Must Now Be Enabled

To determine exactly what modules/themes to enable in `Drupal8` I visited my `mm3.getcybersolutions.com` server and expected to execute a `drush pm-list` command, but `drush` is NOT installed there!




# Default Sites with Config Export/Import

Opened a terminal into the `d8_php` container from Portainer.  From the `wodby@php.container:/var/www/html $` prompt...

```
composer require drupal/install_profile_generator  
composer update
composer remove drupal/install_profile_generator

```
The module was removed in the end because it did NOT work.  Going to try using `drush config:pull` and `drush config:import` instead **only after extending Drupal8 with contributed and custom modules/themes**.











---  
The original *README.md* from *Docker4Drupal* can be found at https://github.com/wodby/docker4drupal/blob/master/README.md.
