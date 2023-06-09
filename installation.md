# Installation guide

Following the guide below should help you make a working starting development environment.

## New repository
Create a new laravel repository inside a parent directory. Installation instructions are available [here](https://laravel.com/docs/9.x/installation#laravel-and-docker).

Be sure to check the [Sail installation subsection](https://laravel.com/docs/9.x/installation#choosing-your-sail-services) to create a repository with sail already added.

The following command will create a new repository and install Sail with dependencies for MySQL and Redis.

```bash
curl -s "https://laravel.build/arm-hrm?with=mysql,redis" | bash
```

### Initializing the repo

If you create a repository through the above example, most of the grunt work will be done for you automatically (copying the .env.example file, installing composer dependencies, ...).

### Modifying the env

You can check the `.env` file and modify the database connection info. Personally we like to add a custom port for Sail through the .env file with the `APP_PORT` variable.

```bash
APP_URL="http://localhost:8080"
APP_PORT="8080"
```

#### Important!

Delete all base laravel migrations from your repository.
The migrations can be found in the `database/migrations` folder. This usually includes the users, password_resets, failed_jobs and personal_access_tokens tables. Not deleting those files will result in issues migrating the starting database.

### ARM package
Download the `arm` project into the same parent directory as your new project repository.

As per the ARM readme file, add the required variables to the project's `.env` file.

Example:
```bash
SESSION_DOMAIN=localhost
SESSION_COOKIE=localhost_test_session
ARM_CURRENT_PROJECT=hrm
ARM_LOGIN="https://arm.si/"
```

set the `SESSION_DRIVER` to `database`

```bash
SESSION_DRIVER=database
```

Add the `arm` project volume to the `docker-compose.yml` file under `volumes:`

```yaml
  volumes:
    - '.:/var/www/html'
    - './../arm:/var/www/arm'
```

Add the arm path to the `composer.json` file, under

```json
"repositories": [
    {
        "type": "path",
        "url": "../arm"
    }
],
```

### Finishing touches and commands

Modify your `User.php` model file, by extending the user with the ARM user class.

```php
<?php

namespace App\Models;

class User extends \ARM\Models\User
{
    //
}
```

The next section will presume you have set up a [shell alias for sail](https://laravel.com/docs/master/sail#configuring-a-shell-alias). If not, use `./vendor/bin/sail` instead of `sail`.

Run the development environment with `sail up`

Require the ARM package through composer with `sail composer require media24si/arm`

Run the following command to create the database: `sail artisan migrate`.

Install the frontend dependencies with `sail npm install` and `sail npm run dev` .

Create a test user either through the database or with tinker through `sail artisan tinker`

```php
User::create(['email' => 'user@example.dev', 'name' => 'Test User', 'password' => \Hash::make('password')]);
```

Login inside the project with the use of this route `/dev/login/{user_id}`. We will provide access to the `login` package and environment, should it become neccessary, but currently we are all using this route to login, since it shortens the dev time, and the route allows you to login as any user, for testing purposes.

## Existing repository

Install the required dependencies through composer with the following command:

```bash
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/var/www/html \
    -v $(pwd)/../arm:/var/www/arm \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```

Copy the `.env.example` file to `.env` with: `cp .env.example .env`.

Download the `arm` project into the same parent directory as your project repository.

The next section will presume you have set up a [shell alias for sail](https://laravel.com/docs/master/sail#configuring-a-shell-alias). If not, use `./vendor/bin/sail` instead of `sail`.

Run the development environment with `sail up`.

Generate a new application key with `sail artisan key:generate`.

Run the following command to create run the database migrations: `sail artisan migrate`.

Install the frontend dependencies with `sail npm install` and `sail npm run dev` .

Create a test user either through the database or with tinker through `sail artisan tinker`

```php
User::create(['email' => 'user@example.dev', 'name' => 'Test User', 'password' => \Hash::make('password')]);
```

Login inside the project with the use of this route `/dev/login/{user_id}`. We will provide access to the `login` package and environment, should it become neccessary, but currently we are all using this route to login, since it shortens the dev time, and the route allows you to login as any user, for testing purposes.

## Install - development - with both login and project

Download both `arm` and `login` packages into the same parent directory as the project package.
The docker compose file expects all directories to be there.

Create a `docker-compose.override.yml` file and paste this inside:
```yml
version: '3'
services:
    laravel.login:
        build:
            context: ./docker/8.2
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        extra_hosts:
            - 'host.docker.internal:host-gateway'
        ports:
            - '${LOGIN_PORT:-80}:80'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - './../login:/var/www/html'
            - './../arm:/var/www/arm'
        networks:
            - sail
        depends_on:
            - mysql
            - redis
```

Run the following command to install composer dependencies with docker inside both `login` and your project folders.
```bash
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/var/www/html \
    -v $(pwd)/../arm:/var/www/arm \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
```

Make sure both the `login` and `arm` apps have the same ENV variables:

- SESSION_DOMAIN
- SESSION_COOKIE
- ARM_LOGIN


## Running the application

### Development

Since default Laravel apps are now bundled with Vite instead of Mix, you need to keep the watcher running for build files to work correctly (through `sail npm run dev`). The Vite ports are already forwarded by default in the docker YAML file.

### Building

`sail npm run build`
