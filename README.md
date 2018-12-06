# Docker setup for Sylius plugin development

## Overview

The goal of this docker setup is to help in the development of Sylius plugins (**Since 1.3**), so at the moment it has a very focused and limited scope. It only requires a basic understanding of docker and docker-compose, but if you face any problem, ask for help in Sylius slack `#docker` channel.

## Services

### Selenium
Selenium support comes out of the box so you only need to configure `behat.yml` as described in setup.

### Xdebug
As Docker assigns subnet in a dynamic fashion, we need to setup a "static" subnet `172.20.0.0/16` under network configuration in `docker-compose.yml`. If you experience some sort of conflict with your own network, just change it to anything like `172.*.0.0/16`. Just remember to change also `PHP_XDEBUG_REMOTE_HOST` to be in the same subnet. It also comes with the asumption that you are using your Xdebug client on port `9000`. If you are using it on another port, just change it in `docker-compose.yml`.

### Mailhog
MailHog is an email testing tool that catches all application emails before they reach the outside world. You can access it at http://localhost:8025.

### Adminer
Adminer is a database management tool. You can access it at http://localhost:8033.

### App
Your app will be accesible via http://localhost. It runs on port `80`. If you already have another service listening in that port, you have to change it in `docker-compose.yml` from `80:80` to `8080:80` or something else.

## Setup
* Ensure `volumes/*` are world writable and `images/*` are world readable.
* Remove `volumes/app/.gitignore`.
* **(Already existing plugin only)** If you already have your plugin, place it in `volumes/app`. It will become `/home/sylius` inside `php` container.
* Fire up containers: `docker-compose up -d`
* Log into app container: `docker exec -i -t php /bin/sh`
* **(New plugin only)** From `/home/sylius` (container workdir) `composer create-project sylius/plugin-skeleton .`
* Change `tests/Application/.env.test.dist` to `.env.test`
* Change `tests/Application/.env.dist` to `.env` **and** `.env.dev`
* Change `behat.yml.dist` to `behat.yml` and add `Behat\MinkExtension`:
```yaml
        Behat\MinkExtension:
            files_path: "%paths.base%/vendor/sylius/sylius/src/Sylius/Behat/Resources/fixtures/"
            base_url: "http://nginx/"
            default_session: symfony
            javascript_session: chrome
            sessions:
                symfony:
                    symfony: ~
                chrome:
                    selenium2:
                        browser: chrome
                        wd_host: "http://selenium-hub:4444/wd/hub"
                        capabilities:
                            browserName: chrome
                            browser: chrome
                            version: ""
                            marionette: null # https://github.com/Behat/MinkExtension/pull/311
                            chrome:
                                switches:
                                    - "start-fullscreen"
                                    - "start-maximized"
                                    - "no-sandbox"
            show_auto: false
```
* To use mailhog, set `disable_delivery: false` in `tests/Application/config/packages/dev/swiftmailer.yaml`
* From `/home/sylius`, run the following commands **(Note that if you already have your plugin you probably can skip yarn and assets steps)** :

**Test setup**
```bash
$ (cd tests/Application && yarn install)
$ (cd tests/Application && yarn build)
$ (cd tests/Application && bin/console assets:install public -e test)

$ (cd tests/Application && bin/console doctrine:database:create -e test)
$ (cd tests/Application && bin/console doctrine:schema:create -e test)
```

**Dev setup**
```bash
$ (cd tests/Application && bin/console doctrine:database:create -e dev)
$ (cd tests/Application && bin/console doctrine:schema:create -e dev)

$ (cd tests/Application && bin/console sylius:fixtures:load -e dev)
```

## Running plugin tests

* Fire up containers: `docker-compose up -d`
* Log into app container: `docker exec -i -t php /bin/sh`
* Execute `switch-env test`

- PHPUnit

```bash
$ vendor/bin/phpunit
```

- PHPSpec

```bash
$ vendor/bin/phpspec run
```

- Behat (non-JS scenarios)

```bash
$ vendor/bin/behat --tags="~@javascript"
```

- Behat (JS scenarios)

```bash
$ vendor/bin/behat --tags="@javascript"
```

## Change environment

For ease of development, there is a shortcut to easily change between dev|test environments, just type

```
switch-env [test|dev]
```
