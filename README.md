# moodle-docker: Docker Containers for Moodle Developers
# Overview
These instructions cover setting up a Moodle (omnibus) development environment. It uses Docker to run Moodle LMS and the related services for a development environment. It is a fork of: https://github.com/moodlehq/moodle-docker but with additions for SSO and Matrix.

The services set up are:
* Moodle LMS
* Postgres Database server
* Mailhog
* Selenium
* Matrix (Both Synapse and Element)
* Keycloak

Once the following steps are complete the sites can be accessed at the following URLs:
* Moodle LMS: https://webserver/
* Keycloak: https://keycloak:8443/
* Element: https://element:8081/
* Synapse: https://synapse:8008/
* Mailhog: http://webserver:1234/_/mail

The entire setup process should take about: 45 minutes

# MacOS Host Setup
These are the steps that need to be done to set up the development environment on a MacOS based machine.
## Homebrew Setup
Home brew is a package manager for OSX, it makes it easier to install things.<br/>
To add it to a Mac:<br/>
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

There are some helpful utilities we want to install initially:<br/>
`brew install libpq`
`echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc`

Next install openssl on mac using homebrew, run the following command:<br/>
`brew install openssl`

## Docker Setup
Best place to go for both intel and apple silicon is here: https://docs.docker.com/desktop/install/mac-install/

On Mac we need to enable a couple of experimental settings to improve performance.<br/>
In Docker desktop on Mac:
* Go to settings > General
* Enable: Use virtualization framework
* Enable: VirtioFS accelerated directory sharing
* Restart the host machine.

## PHP
We use homebrew to install the required versions of PHP we need for development on the host machine.<br/>
`brew install php@7.4`<br/>
`brew install php@8.0`

The php.ini and php-fpm.ini file can be found in:<br/>
`/usr/local/etc/php/7.4/`

Switch from 7.4 to 8.0:<br/>
`brew unlink php@7.4`<br/>
`brew link php@8.0 --force`

## Moodle LMS Code
The following steps are required to get the Moodle code locally and initial setups.<br/>
Start by cloning the Moodle codebase locally to the host:</br>
`git clone https://github.com/moodle/moodle.git`

You can then checkout any branch you want to work with.

## Node Setup
Node is required for js compilation etc.<br/>
First we install NVM:
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash`

Then we set up node, run the following:<br/>
`cd moodle_local #Or the location where you cloned the Moodle code`<br/>
`nvm install`<br/>
`nvm use`

nvm use will also output the node version, use it in the following command:<br/>
`nvm alias default v16.17.0`

Next run the package install:<br/>
`npm install`

# Moodle Docker
Moodle HQ have been awesome enough to create a set of Docker containers and a Docker compose setup that automates much of the setup and configuration of running Moodle in Docker.

We’ve further extended this work with a “Custom setup” that pre-builds some of the tools and setup options we use in our development workflow. This document contains instructions on how to use both methods.

The aim of the custom setup is to decrease the setup time for a Moodle development workflow and to have commonly used components pre-built.

## Initial steps
We start by cloning our fork of the Moodle Docker repository:<br/>
`git clone git@github.com:mattporritt/moodle-docker.git`

Checkout out the branch: `omnibus`

Add a sample config.php file to Moodle code:<br/>
`cp moodle-docker/config.docker-template.php moodle_local/config.php`

<<<<<<< HEAD
## Use containers for running behat tests

```bash
# Initialize behat environment
bin/moodle-docker-compose exec webserver php admin/tool/behat/cli/init.php
# [..]

# Run behat tests
bin/moodle-docker-compose exec -u www-data webserver php admin/tool/behat/cli/run.php --tags=@auth_manual
Running single behat site:
Moodle 3.4dev (Build: 20171006), 33a3ec7c9378e64c6f15c688a3c68a39114aa29d
Php: 7.1.9, pgsql: 9.6.5, OS: Linux 4.9.49-moby x86_64
Server OS "Linux", Browser: "firefox"
Started at 25-05-2017, 19:04
...............

2 scenarios (2 passed)
15 steps (15 passed)
1m35.32s (41.60Mb)
```

Notes:

* The behat faildump directory is exposed at http://localhost:8000/_/faildumps/.
* Use `MOODLE_DOCKER_BROWSER` to switch the browser you want to run the test against.
  You need to recreate your containers using `bin/moodle-docker-compose` as described below, if you change it.

## Use containers for running phpunit tests

```bash
# Initialize phpunit environment
bin/moodle-docker-compose exec webserver php admin/tool/phpunit/cli/init.php
# [..]

# Run phpunit tests

bin/moodle-docker-compose exec webserver vendor/bin/phpunit auth/manual/tests/manual_test.php
Moodle 4.0.4 (Build: 20220912), ef7a51dcb8e805a6889974b04d3154ba8bd874f2
Php: 7.3.33, pgsql: 11.15 (Debian 11.15-1.pgdg90+1), OS: Linux 5.10.0-11-amd64 x86_64
PHPUnit 9.5.13 by Sebastian Bergmann and contributors.

..                                                                  2 / 2 (100%)

Time: 00:00.304, Memory: 72.50 MB

OK (2 tests, 7 assertions)
```

Notes:
* If you want to run tests with code coverage reports:
```
# Build component configuration
bin/moodle-docker-compose exec webserver php admin/tool/phpunit/cli/util.php --buildcomponentconfigs
# Execute tests for component
bin/moodle-docker-compose exec webserver php -d pcov.enabled=1 -d pcov.directory=. vendor/bin/phpunit --configuration reportbuilder --coverage-text
```
* See available [Command-Line Options](https://phpunit.readthedocs.io/en/9.5/textui.html#textui-clioptions) for further info

## Use containers for manual testing

```bash
# Initialize Moodle database for manual testing
bin/moodle-docker-compose exec webserver php admin/cli/install_database.php --agree-license --fullname="Docker moodle" --shortname="docker_moodle" --summary="Docker moodle site" --adminpass="test" --adminemail="admin@example.com"
```

Notes:
* Moodle is configured to listen on `http://localhost:8000/`.
* Mailhog is listening on `http://localhost:8000/_/mail` to view emails which Moodle has sent out.
* The admin `username` you need to use for logging in is `admin` by default. You can customize it by passing `--adminuser='myusername'`

## Use containers for running behat tests for the Moodle App

In order to run Behat tests for the Moodle App, you need to install the [local_moodlemobileapp](https://github.com/moodlehq/moodle-local_moodlemobileapp) plugin in your Moodle site. Everything else should be the same as running standard Behat tests for Moodle. Make sure to filter tests using the `@app` tag.

The Behat tests will be run against a container serving the mobile application, you have two options here:

1. Use a Docker image that includes the application code. You need to specify the `MOODLE_DOCKER_APP_VERSION` env variable and the [moodlehq/moodleapp](https://hub.docker.com/r/moodlehq/moodleapp) image will be downloaded from Docker Hub. You can read about the available images in [Moodle App Docker Images](https://docs.moodle.org/dev/Moodle_App_Docker_Images) (for Behat, you'll want to run the ones with the `-test` suffix).

2. Use a local copy of the application code and serve it through Docker, similar to how the Moodle site is being served. Set the `MOODLE_DOCKER_APP_PATH` env variable to the codebase in you file system. This will assume that you've already initialized the app calling `npm install` and `npm run setup` locally.

For both options, you also need to set `MOODLE_DOCKER_BROWSER` to "chrome".

```bash
# Install local_moodlemobileapp plugin
git clone https://github.com/moodlehq/moodle-local_moodlemobileapp "$MOODLE_DOCKER_WWWROOT/local/moodlemobileapp"

# Initialize behat environment
bin/moodle-docker-compose exec webserver php admin/tool/behat/cli/init.php
# (you should see "Configured app tests for version X.X.X" here)

# Run behat tests
bin/moodle-docker-compose exec -u www-data webserver php admin/tool/behat/cli/run.php --tags="@app&&@mod_login"
Running single behat site:
Moodle 4.0dev (Build: 20200615), a2b286ce176fbe361f0889abc8f30f043cd664ae
Php: 7.2.30, pgsql: 11.8 (Debian 11.8-1.pgdg90+1), OS: Linux 5.3.0-61-generic x86_64
Server OS "Linux", Browser: "chrome"
Browser specific fixes have been applied. See http://docs.moodle.org/dev/Acceptance_testing#Browser_specific_fixes
Started at 13-07-2020, 18:34
.....................................................................

4 scenarios (4 passed)
69 steps (69 passed)
3m3.17s (55.02Mb)
```

If you are going with the second option, this *can* be used for local development of the Moodle App, given that the `moodleapp` container serves the app on the local 8100 port. However, this is intended to run Behat tests that require interacting with a local Moodle environment. Normal development should be easier calling `npm start` in the host system.

By all means, if you don't want to have npm installed locally you can go full Docker executing the following commands before starting the containers:

```
docker run --volume $MOODLE_DOCKER_APP_PATH:/app --workdir /app bash -c "npm install npm@7 -g && npm ci"
```

You can learn more about writing tests for the app in [Acceptance testing for the Moodle App](https://docs.moodle.org/dev/Acceptance_testing_for_the_Moodle_App).

## Using VNC to view behat tests

If `MOODLE_DOCKER_SELENIUM_VNC_PORT` is defined, selenium will expose a VNC session on the port specified so behat tests can be viewed in progress.

For example, if you set `MOODLE_DOCKER_SELENIUM_VNC_PORT` to 5900..
1. Download a VNC client: https://www.realvnc.com/en/connect/download/viewer/
2. With the containers running, enter 0.0.0.0:5900 as the port in VNC Viewer. You will be prompted for a password. The password is 'secret'.
3. You should be able to see an empty Desktop. When you run any Behat tests a browser will popup and you will see the tests execute.

## Stop and restart containers

`bin/moodle-docker-compose down` which was used above after using the containers stops and destroys the containers. If you want to use your containers continuously for manual testing or development without starting them up from scratch everytime you use them, you can also just stop without destroying them. With this approach, you can restart your containers sometime later, they will keep their data and won't be destroyed completely until you run `bin/moodle-docker-compose down`.

```bash
# Stop containers
bin/moodle-docker-compose stop

# Restart containers
bin/moodle-docker-compose start
```

## Environment variables

You can change the configuration of the docker images by setting various environment variables **before** calling `bin/moodle-docker-compose up`.
When you change them, use `bin/moodle-docker-compose down && bin/moodle-docker-compose up -d` to recreate your environment.

| Environment Variable                      | Mandatory | Allowed values                        | Default value | Notes                                                                        |
|-------------------------------------------|-----------|---------------------------------------|---------------|------------------------------------------------------------------------------|
| `MOODLE_DOCKER_DB`                        | yes       | pgsql, mariadb, mysql, mssql, oracle  | none          | The database server to run against                                           |
| `MOODLE_DOCKER_WWWROOT`                   | yes       | path on your file system              | none          | The path to the Moodle codebase you intend to test                           |
| `MOODLE_DOCKER_DB_VERSION`                | no        | Docker tag - see relevant database page on docker-hub | mysql: 8.0 <br/>pgsql: 13 <br/>mariadb: 10.7 <br/>mssql: 2017-latest <br/>oracle: 21| The database server docker image tag |
| `MOODLE_DOCKER_PHP_VERSION`               | no        | 8.1, 8.0, 7.4, 7.3, 7.2, 7.1, 7.0, 5.6     | 8.0           | The php version to use                                                       |
| `MOODLE_DOCKER_BROWSER`                   | no        | firefox, chrome,  firefox:&lt;tag&gt;, chrome:&lt;tag&gt; | firefox:3       | The browser to run Behat against. Supports a colon notation to specify a specific Selenium docker image version to use. e.g. firefox:2.53.1 can be used to run with older versions of Moodle (<3.5)              |
| `MOODLE_DOCKER_PHPUNIT_EXTERNAL_SERVICES` | no        | any value                             | not set       | If set, dependencies for memcached, redis, solr, and openldap are added      |
| `MOODLE_DOCKER_BEHAT_FAILDUMP`            | no        | Path on your file system              | not set       | Behat faildumps are already available at http://localhost:8000/_/faildumps/ by default, this allows for mapping a specific filesystem folder to retrieve the faildumps in bulk / automated ways |
| `MOODLE_DOCKER_DB_PORT`                   | no        | any integer value                     | none          | If you want to bind to any host IP different from the default 127.0.0.1, you can specify it with the bind_ip:port format (0.0.0.0 means bind to all). Username is "moodle" (or "sa" for mssql) and password is "m@0dl3ing". |
| `MOODLE_DOCKER_WEB_HOST`                  | no        | any valid hostname                    | localhost     | The hostname for web                                |
| `MOODLE_DOCKER_WEB_PORT`                  | no        | any integer value (or bind_ip:integer)| 127.0.0.1:8000| The port number for web. If set to 0, no port is used.<br/>If you want to bind to any host IP different from the default 127.0.0.1, you can specify it with the bind_ip:port format (0.0.0.0 means bind to all) |
| `MOODLE_DOCKER_SELENIUM_VNC_PORT`         | no        | any integer value (or bind_ip:integer)| not set       | If set, the selenium node will expose a vnc session on the port specified. Similar to MOODLE_DOCKER_WEB_PORT, you can optionally define the host IP to bind to. If you just set the port, VNC binds to 127.0.0.1 |
| `MOODLE_DOCKER_APP_PATH`                  | no        | path on your file system              | not set       | If set and the chrome browser is selected, it will start an instance of the Moodle app from your local codebase |
| `MOODLE_DOCKER_APP_VERSION`               | no        | a valid [app docker image version](https://docs.moodle.org/dev/Moodle_App_Docker_images) | not set       | If set will start an instance of the Moodle app if the chrome browser is selected |
| `MOODLE_DOCKER_APP_RUNTIME`               | no        | 'ionic3' or 'ionic5'                  | not set       | Set this to indicate the runtime being used in the Moodle app. In most cases, this can be ignored because the runtime is guessed automatically (except on Windows using the `.cmd` binary). In case you need to set it manually and you're not sure which one it is, versions 3.9.5 and later should be using Ionic 5. |
| `MOODLE_DOCKER_APP_NODE_VERSION`          | no        | [node](https://hub.docker.com/_/node) image version tag                | not set       | Node version to run the app. In most cases, this can be ignored because the version is parsed from the project's `.nvmrc` file. This will only be used when the runtime is `ionic5` and the app is running from the local filesystem. |

## Local customisations

In some situations you may wish to add local customisations, such as including additional containers, or changing existing containers.

This can be accomplished by specifying a `local.yml`, which will be added in and loaded with the existing yml configuration files automatically. For example:

``` file="local.yml"
version: "2"
services:

  # Add the adminer image at the latest tag on port 8080:8080
  adminer:
    image: adminer:latest
    restart: always
    ports:
      - 8080:8080
    depends_on:
      - "db"

  # Modify the webserver image to add another volume:
  webserver:
    volumes:
      - "/opt/data:/opt/data:cached"
```

## Using XDebug for live debugging

The XDebug PHP Extension is not included in this setup and there are reasons not to include it by default.

However, if you want to work with XDebug, especially for live debugging, you can add XDebug to a running webserver container easily:

```
# Install XDebug extension with PECL
moodle-docker-compose exec webserver pecl install xdebug

# Set some wise setting for live debugging - change this as needed
read -r -d '' conf <<'EOF'
; Settings for Xdebug Docker configuration
xdebug.mode = debug
xdebug.client_host = host.docker.internal
EOF
moodle-docker-compose exec webserver bash -c "echo '$conf' >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini"

# Enable XDebug extension in Apache and restart the webserver container
moodle-docker-compose exec webserver docker-php-ext-enable xdebug
moodle-docker-compose restart webserver
```

While setting these XDebug settings depending on your local need, please take special care of the value of `xdebug.client_host` which is needed to connect from the container to the host. The given value `host.docker.internal` is a special DNS name for this purpose within Docker for Windows and Docker for Mac. If you are running on another Docker environment, you might want to try the value `localhost` instead or even set the hostname/IP of the host directly.

After these commands, XDebug ist enabled and ready to be used in the webserver container.
If you want to disable and re-enable XDebug during the lifetime of the webserver container, you can achieve this with these additional commands:

```
# Disable XDebug extension in Apache and restart the webserver container
moodle-docker-compose exec webserver sed -i 's/^zend_extension=/; zend_extension=/' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
moodle-docker-compose restart webserver

# Enable XDebug extension in Apache and restart the webserver container
moodle-docker-compose exec webserver sed -i 's/^; zend_extension=/zend_extension=/' /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
moodle-docker-compose restart webserver
```

## Advanced usage

As can be seen in [bin/moodle-docker-compose](https://github.com/moodlehq/moodle-docker/blob/master/bin/moodle-docker-compose),
this repo is just a series of Docker Compose configurations and light wrapper which make use of companion docker images. Each part
is designed to be reusable and you are encouraged to use the docker [compose] commands as needed.

## Companion docker images

The following Moodle customised docker images are close companions of this project:

* [moodle-php-apache](https://github.com/moodlehq/moodle-php-apache): Apache/PHP Environment preconfigured for all Moodle environments
* [moodle-db-mssql](https://github.com/moodlehq/moodle-db-mssql): Microsoft SQL Server for Linux configured for Moodle
* [moodle-db-oracle](https://github.com/moodlehq/moodle-db-oracle): Oracle XE configured for Moodle

## Contributions

Are extremely welcome!
=======
Copy the sample environment file:<br/>
`cp moodle-docker/.env.example moodle-docker/.env`

Update the MOODLE_DOCKER_WWWROOT variable in the .env file to the location of the Moodle code on the host.

## Host file
To make it easier to access the sites that have been setup and to allow correct SSO workflows, the hosts file on the machine running the docker containers needs to be updated. With extra services added to the localhost entry.

Update your local hosts file: `/etc/hosts` to include the following names for the localhost IP (127.0.0.1):
* Keycloak
* Synapse
* Element
* Webserver (Moodle)

If you haven’t already customised your hosts file the line should look like:<br/>
`127.0.0.1   	localhost synapse webserver keycloak element`

## SSL Setup
Next make the self signed certs so we can run Moodle and associated services over ssl/tls. This is required to setup SSO.<br/>
There is a helper script that creates the root CA and dev certificates and keys.

Change to the directory of the script first:<br/>
`cd moodle-docker/moodle_dev/assets/certs`

Then run it for each service that we require certificates for. The first time the script is run it will generate a root cert and key. After that it will re-use the root cert and just make the certs required for each service we want to run using SSL/TLS.

Run the script:<br/>
`./createcerts.sh webserver`

Then run it for each additional service we need<br/>
`./createcerts.sh keycloak`
`./createcerts.sh synapse`
`./createcerts.sh element`

These certs will be automatically loaded into the containers. We only need to generate them.

Next we add the root CA, this is to remove the browser self signed errors.<br/>
To add the root CA to the Mac OS keychain. Run this on this host machine:<br/>
`sudo security add-trusted-cert -d -r trustRoot -k "/Library/Keychains/System.keychain" ca.pem`

To add the certificate to firefox to get rid of the self signed warning. (Chrome should “just work”):<br/>
1. Run Firefox.
2. Position to Settings > Privacy & Security.
3. Under Certificates, click "View Certificates..."
4. In the Certificate Manager, click the Authorities tab.
5. Click the Import button to import your certificate.
6. You might be prompted to set the trust level upon importing the certificate. ...
7. Restart Firefox.

## Build and install
Next we need to build our version of the moodle dev container:<br/>
`cd moodle-docker/moodle_dev`<br/>
`docker build -t "mattp:moodle_dev" .`

Finally actually start the services:<br/>
`cd moodle-docker/bin`<br/>
`./moodle-docker-compose up -d`

Next we need to install Moodle:<br/>
`./moodle-docker-compose exec webserver php admin/cli/install_database.php --agree-license --fullname="Moodle Master" --shortname="docker_moodle" --summary="Moodle dev site" --adminpass="test" --adminemail="you@gmail.com"`

# PHPStorm Setup
Next we set up profiling in PHPStorm. Go to your PhpStorm and go to:<br/>
`Run -> Edit configurations`<br/>
and select new:<br/>
`PHP Remote Debug`

`Name: "xdebug webserver" (or what you want to)`<br/>
`Configuration: check "Filter debug connection by IDE key"`<br/>
`IDE key(session id): "phpstorm"`<br/>
`Define a new server:`<br/>
`Name: must be "moodle-local"`<br/>
`Host: webserver`<br/>
`Port: Must be the port you're using for the web server. This should be 443`<br/>
`Debugger: use the default (Xdebug)`<br/>
`Check "Use path mappings (...)"`<br/>
`Set for your "Project files" Moodle root the "Absolute path on the server" as "/var/www/html"`<br/>
`Apply and OK on this screen. This screen will be closed.`<br/>
`Apply and OK on the next screen. Settings screen will be closed.`<br/>

Now, test that live debugging works. To do so:<br/>
Put a breakpoint on /index.php file.<br/>
Press telephone icon with a red symbol with title "Start listening for PHP Debug Connections": telephone should appear with some waves now.<br/>

Finally we need to add the xdebug browser extension to Firefox. Go here to install the extension:<br/>
https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/

Once installed you can enable and disable debugging from Firefox (providing PHP storm is listening.

# Keycloak IdP Setup
This will set up Keycloak as an Identity Provider (IdP) for Moodle and Synapse. Users will be able to log into Moodle and Element via Keycloak once the following configuration is complete.

## Keycloak
The following steps will set up a Moodle LMS client in Keycloak so users can authenticate to Moodle from Keycloak using OIDC/Oauth.

Access the Moodle realm in Keycloak:<br/>
* First log into Keycloak (https://keycloak:8443/ ) using the admin credentials you defined in the .env file in the root of the moodle-docker project.
* Then click on the link to Keycloak Administration
* Then on the left of the screen change the “Realm Select” drop down menu from master to moodle.

Set up the Moodle client:<br/>
* Click Clients from the Manage menu on the left of the page
* From the list of clients that are displayed click on the “moodle-client” link in the Client ID column
* Click the Credentials tab
* Click the Regenerate button for the Client secret
* Note the Client secret.

Set up the Synapse client:<br/>
* Click Clients from the Manage menu on the left of the page
* From the list of clients that are displayed click on the “moodle-client” link in the Client ID column
* Click the Credentials tab
* Click the Regenerate button for the Client secret
* Note the Client secret.

We also need to create at least one user in the moodle realm in keycloak. All users that log into Moodle using SSO via Keycloak need an account in the moodle realm.<br/>
To do this:<br/>
* Click Users from the Manage menu on the left of the page
* Click the Add user button
* Set the following settings for the new user:
  - Username
  - Email (can be fake)
  - Set Email verified to true/on
  - First name
  - Last name
* Click the create button

## Moodle
The following steps will set up Moodle LMS as a Service Provider with  Keycloak as an Identity Provider.

As we are using a development environment we need to allow non standard port.<br/>
To do this:<br/>
* Log into the Moodle LMS instance (https://webserver) as an admin.
* Access the HTTP security settings: Site administration > General > HTTP security (https://webserver/admin/settings.php?section=httpsecurity)
* In the “cURL allowed ports list” add the port: 8443 (We are using non standard ports in development)
* Click “Save changes”

Next we need to set up the Oauth2 service for Keycloak in Moodle LMS:<br/>
* Log into the Moodle LMS instance (https://webserver) as an admin.
* Access the OAuth2 services settings: Site administration > Server > OAuth2 services (https://webserver/admin/tool/oauth2/issuers.php )
* Click the “Custom” button for the “Create new service” setting
* Set the following settings:
  - Name to: Keycloak
  - Client ID to: moodle-client
  - Client secret to: the value you noted from keycloak during setup
  - Service base URL to: https://keycloak:8443/realms/moodle/
  - Logo URL to: https://keycloak:8443/resources/u40ce/login/keycloak/img/favicon.ico
  - This service will be used to: Login page and internal services
  - Unselect: Require email validation
  - Select: I understand that disabling email verification can be a security issue.
* Click: Save changes

Next we configure the fields from Keycloak against the Moodle user profile:<br/>
* From the “Edit” column for the Keycloak “service”, click the “Configure user field mappings” icon
* Click the “Create new user field mapping for issuer ‘Keycloak’” button
* Set the following settings
  - External field name to: preferred_username
  - Internal field name  to: username

Finally, we configure the Oauth2 authentication plugin to allow users to log into Moodle LMS using Keycloak:<br/>
* Log into the Moodle LMS instance (https://webserver) as an admin.
* Access the Manage authentication settings: Site administration > Plugins > Authentication >  Manage authentication (https://webserver/admin/settings.php?section=manageauths )
* Enable the Oauth2 authentication plugin
Keycloak users will now be able to use Keycloak SSO to log into Moodle.

## Synapse
The following steps will set up Synapse as a Service Provider with Keycloak as an Identity Provider.

To do this:<br/>
* Open the homeserver.yaml for the synapse configuration: moodle-docker/synapse_data/homeserver.yaml
* Change the value for client_secret to the client secret value for the synapse-client from Keycloak
* Restart the synapse container in Docker.

# Accessing Sites
Once the above steps are complete the sites can be accessed at the following URLs:
* Moodle LMS: https://webserver/
* Keycloak: https://keycloak:8443/
* Element: https://element:8081/
* Synapse: https://synapse:8008/
* Mailhog: http://webserver:1234/_/mail

# PHP Unit Tests
Running unit tests in the docker container is very similar to running them from the command line in a VM.
To initialise phpunit environment:<br/>
`cd bin`<br/>
`./moodle-docker-compose exec webserver php admin/tool/phpunit/cli/init.php`

To run phpunit tests:<br/>
`./moodle-docker-compose exec webserver vendor/bin/phpunit`

# BEHAT Tests
Running behat tests in the docker container is very similar to running them from the command line in a VM.<br/>
To initialise behat environment:<br/>
`./moodle-docker-compose exec webserver php admin/tool/behat/cli/init.php`

To run behat tests:<br/>
`./moodle-docker-compose exec -u www-data webserver php admin/tool/behat/cli/run.php --tags=@auth_manual`

# Mailhog
MailHog is an email-testing tool with a fake SMTP server underneath. It encapsulates the SMTP protocol with extensions and does not require specific backend implementations. MailHog runs a super simple SMTP server that hogs outgoing emails sent to it. You can see the hogged emails in a web interface.

To access mailhog:<br/>
http://webserver:1234/_/mail

# Useful Docker Commands
To access the container directly:<br/>
`./moodle-docker-compose exec -it webserver /bin/bash`

To get container logs (which include apache logs for webserver container:<br/>
`docker logs -f moodlemaster-webserver-1`

To shut things down:<br/>
`./moodle-docker-compose down`

To access the Moodle DB from the host machine:<br/>
`psql -h 127.0.0.1 -p 5433 -U moodle`

To dump the db from the host machine:<br/>
`pg_dump -h 127.0.0.1 -p 5433 -U moodle -Fc moodle > moodle.dump`

To restore the db when none exists:<br/>
`pg_restore -h 127.0.0.1 -p 5433 -U moodle -d moodle moodle.dump`
>>>>>>> d1b4c76586 (Update Readme specific to the omnibus branch)
