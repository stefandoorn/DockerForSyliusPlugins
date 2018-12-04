# Docker setup for Sylius plugin development

## Installation

* Ensure `volumes/*` are world writable and `images/*` are world readable.
* Remove `volumes/app/.gitignore`
* Change `.env.test.dist` to `.env`
* Fire up containers: `docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d`
* Log into app container: `docker exec -i -t php /bin/sh`
* From `/home/sylius` (container workdir) run `composer create-project sylius/plugin-skeleton .`
* In `tests/Application/config/packages/doctrine.yaml` set `server_version: 'mariadb-10.4.0'`
* In `behat.yml` file, add `Behat\MinkExtension`:
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
* From `/home/sylius`, run the following commands:
 ```bash
$ (cd tests/Application && yarn install)
$ (cd tests/Application && yarn build)
$ (cd tests/Application && bin/console assets:install public -e test)

$ (cd tests/Application && bin/console doctrine:database:create -e test)
$ (cd tests/Application && bin/console doctrine:schema:create -e test)
```

## Running plugin tests

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

## Development environment
* Change `.env.dev.dist` to `.env`
* Fire up containers: `docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d`
* Log into app container: `docker exec -i -t php /bin/sh`
* From `/home/sylius`, run the following commands:
 ```bash
$ (cd tests/Application && bin/console doctrine:database:create -e dev)
$ (cd tests/Application && bin/console doctrine:schema:create -e dev)

$ (cd tests/Application && bin/console sylius:fixtures:load -e dev)
```
* To use mailhog, set `disable_delivery: false` in `tests/Application/config/packages/dev/swiftmailer.yaml`
* Adminer: `http://localhost:8033`
* Mailhog: `http://localhost:8025`
* App: `http://localhost`
