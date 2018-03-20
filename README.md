[![Build Status](https://travis-ci.org/fmfpereira/dlamp.svg?branch=master)](https://travis-ci.org/fmfpereira/dlamp)

# README

dLAMP is a Docker Compose configuration to run LAMP development stack containers.

## Table of Contents

- [Background](#background)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Known limitations](#known-limitations)
- [Support](#support)
- [Contributing](#contributing)

## Background

dLAMP was created to build a simple solution for LAMP development environments. It provides a solid starting point which can be optimized for your needs.

With dLAMP you will orchestrate two containers:

- [PHP/Apache](https://hub.docker.com/_/php) with XDebug.
- [MySQL (Percona)](https://hub.docker.com/_/percona).

Project structure:
- conf (project configuration folder)
  - php.ini (overrides default PHP configuration)
  - my.cnf (overrides default MySQL configuration)
  - apache.conf (overrides default Apache VirtualHost)
- data (project data folder)
  - apache (parent docroot folder)
     - web (docroot folder)
  - mysql
     - import (automatically import MySQL dumps files when the container named volume is created)
- docker-compose.yml (Docker Compose file)
- .docker (Docker files)
  - php (PHP Dockerfile folder)

## Requirements
- docker
- docker-compose

## Installation

- [Download](https://github.com/fmfpereira/dlamp/releases) or [clone](https://github.com/fmfpereira/dlamp.git) the project.

- **Note:** xdebug is not yet compatible with PHP7.2, the latest release from official Docker PHP image. We recommend to use DOCKER_PHP_VERSION=7.1 
until a stable release is launched.

- Edit docker-compose.yml file.
    - Select PHP version:
        - dLAMP uses official PHP images. To see available PHP versions check "version"-apache tags at [Docker hub](https://hub.docker.com/_/php/) and replace DOCKER_PHP_VERSION ARG.
        - Ex. PHP 7 DOCKER_PHP_VERSION=7. For PHP 5 use DOCKER_PHP_VERSION=5
            ```yml
            # Docker compose services.
            services:
              # Docker web service.
              web:
                # Configuration options that are applied at build time.
                build: 
                  # Using Dockerfile
                  context: .docker/php
                  # Overrides build arguments, which are environment variables accessible only during the build process.
                  args:
                    - DOCKER_PHP_VERSION=7
            ```    
    - Select which user and group will be used for the Apache process:
        - When using Docker for Linux the file owner uid and gid will be shared across both host and container.
            - By default, Apache uses the uid and gid 33 (www-data). 
            - When you create a file on the container it will use the same uid and gid on host system.
            - You need to set your host user uid and gid on the container to avoid permissions problems.
        - You can leave the default settings if you use Docker for Windows or Docker for Mac.
        ```yml
        # Overrides build arguments, which are environment variables accessible only during the build process.
        args:
          - DOCKER_PHP_VERSION=7
          - DOCKER_APACHE_RUN_USER=www-data
          - DOCKER_APACHE_RUN_GROUP=www-data
          - DOCKER_APACHE_RUN_UID=33
          - DOCKER_APACHE_RUN_GID=33
        ```
    - Select database name and credentials:
        ```yml
        # Add environment variables.
        # Percona environment variables. 
        # Check https://hub.docker.com/_/percona/ for more information.
        environment:
          - MYSQL_ROOT_PASSWORD=db
          - MYSQL_DATABASE=db
          - MYSQL_USER=db
          - MYSQL_PASSWORD=db
        ```
    - Select container exposed ports:
        - web container:
            ```yml
            web:
              # Expose ports.
              ports:
             - "80"
            ```
        - db container:
            ```yml
            db:
              # Expose ports.
              ports:
             - "3306"
            ```      
        - You can either specify both ports (HOST:CONTAINER), or just the container port (a random host port will be chosen).
            - You can check the mapped ports after starting a container:
                ```sh
                [docker-user@docker-host dummy]$ docker ps
                CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                     NAMES
                ab1aeeb4ad5a        dummy_web            "docker-php-entryp..."   17 seconds ago      Up 16 seconds       0.0.0.0:80->80/tcp        dummy_web_1
                1bac80458a96        percona              "docker-entrypoint..."   18 seconds ago      Up 17 seconds       0.0.0.0:32769->3306/tcp   dummy_db_1
                ```
            - In this configuration the web container has been explicitly mapped to port 80 and the db container randomly mapped to port 32769.
        - Each port mapping on the host must be unique per running container. If you need to explicitly map ports on the host, each host port must be unique per running container.
            - Ex. You can explicitly map container port 80 (WWW) to host port 80, but you must be sure that no other host process or container is running on the port.
            - Ex. If you need to run two dLAMP projects at the same time, either leave the default options to generate the host ports randomly or explicitly map different unused ports.

    - If you want to know more details about docker-compose.yml configuration options, check the [Docker Compose file version 3 reference](https://docs.docker.com/compose/compose-file/).

- Create the Docker image:
```sh
[docker-user@docker-host dummy]$ docker-compose build
```
    
## Usage

- Run the containers:
    ```sh
    [docker-user@docker-host dummy]$ docker-compose up -d
    ```

- Attach bash shell to docker container:
    ```sh
    [docker-user@docker-host dummy]$ docker ps
    CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                     NAMES
    ab1aeeb4ad5a        dummy_web            "docker-php-entryp..."   17 seconds ago      Up 16 seconds       0.0.0.0:80->80/tcp        dummy_web_1
    1bac80458a96        percona              "docker-entrypoint..."   18 seconds ago      Up 17 seconds       0.0.0.0:32769->3306/tcp   dummy_db_1
    [docker-user@docker-host dummy]$ docker exec -u www-data -ti ab1 /bin/bash
    ```

    - If you are using Linux, replace www-data for the username set on docker-compose.yml.
    - If you are using MacOS or Windows you can ignore the -u username option.


- Destroy the containers:
    ```sh
    [docker-user@docker-host dummy]$ docker-compose down
    ```

- When you destroy the containers the MySQL data volume will not be removed. To remove the volume:
    ```sh
    [docker-user@docker-host dummy]$ docker volume ls
    DRIVER              VOLUME NAME
    local               2410b28798dd403b26456ba9e3feed27ae642e404eac7a4ff67f0096318f0465
    local               ef6d9e0e1b05740be7973a8ffd0cfbbe3ad9aa1ccbcb7f0ac274f6145a603d9a
    local               dummy_percona_data
    [docker-user@docker-host dummy]$ docker volume rm dummy_percona_data
    ```

- To import the database dumps automatically:
    - The MySQL data volume should not exist:
        - You are running the containers for the first time.
        - You removed the MySQL data volume.
    - Place your .sh, .sql and .sql.gz files on the host mysql import folder.
    - Run the containers.

- MySQL and Apache/PHP containers are different hosts.
    - To connect from Apache/PHP container to MySQL container you should replace "localhost" to "db" (the MySQL hostname in the Docker network) 

## Known limitations

- General
    - Apache process user will be created and assigned at build time. Each time you change any DOCKER_APACHE_RUN_* variable you must re-build the image.

- Windows
    - If you use this project on other path rather than C:\Users (Ex. E:\\) the containers may not start.
        - You should add the path to Shared Folders on Docker options, if available.
        - If you are still struggling to run the project, try to run it on any C:\Users sub-folder.

- MacOS (Docker4Mac)
    - There is a known issue around slow performance using shared volumes on Docker for Mac.
    - If you have any performance issues please read the official [Docker Documentation](https://docs.docker.com/docker-for-mac/osxfs-caching/).
## Support

Please [open an issue](https://github.com/fmfpereira/dlamp/issues/new) for support.

## Contributing

Please contribute using [Github Flow](https://guides.github.com/introduction/flow/). Create a branch, add commits, and [open a pull request](https://github.com/fmfpereira/dlamp/compare).
