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

dLAMP was created to build a simple solution for LAMP development environments. It provides a solid starting point which you can optimize for your needs.

With dLamp you will orchestrate two containers:

* [PHP/Apache](https://hub.docker.com/_/php) with XDebug.
* [MySQL (Percona)](https://hub.docker.com/_/percona).

Project directory structure:
* conf (configuration files).
* data (Apache docroot and Mysql dump importer).
* docker-compose.yml (Docker Compose file)
* .docker (Docker files)

The following container files/directories will be shared within your host:

* Configuration files
    * php.ini (overrides default PHP configuration).
    * my.cnf (overrides default MySQL configuration).
    * apache-default.conf (overrides default Apache configuration).

* Directories
    * /var/www/html (shares docroot with host).
    * /var/lib/mysql (stores data files on host via named volumes).
    * /docker-entrypoint-initdb.d (automatically import MySQL dumps when the container named volume is created).

## Requirements
- Docker

## Installation

- [Download](https://github.com/fmfpereira/dlamp/releases) or [clone](https://github.com/fmfpereira/dlamp.git) the project.

- Edit docker-compose.yml file.
    - Select PHP version: services -> web-> build -> context.
        - Currently dLAMP supports php5.6 and php7:
            - php5.6 uses Dockerfile .docker/php5.6
            - php7 uses Dockerfile .docker/php7
    - Select which user/uid and group/gid will be used for the Apache process: services -> web -> build -> arguments.
        - DOCKER_APACHE_RUN_USER, DOCKER_APACHE_RUN_GROUP, DOCKER_APACHE_RUN_UID, DOCKER_APACHE_RUN_GID
        - When using Docker for Linux the file owner uid and gid will be shared across both host and container.
            - By default, Apache uses the uid and gid 33 (www-data). 
            - When you create a file on the container it will use the same uid and gid on host system.
            - You need to set your host user uid and gid on the container to avoid permissions problems.
    - Select database name and credentials on db -> environment.
        - MYSQL_ROOT_PASSWORD, MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD
    - If you want to know more details about docker-compose.yml configuration options, check the [Docker Compose file version 3 reference](https://docs.docker.com/compose/compose-file/).

- Create the Docker image
```sh
docker-compose build
```
    
## Usage

- Run the containers:
    ```sh
    docker-compose up -d
    ```

- Attach shell to docker container
    ```sh
    docker exec -u "username" -ti "apache/php container hash" /bin/bash
    ```

- Destroy the containers:
    ```sh
    docker-compose down
    ```

- When you destroy the containers the MySQL data volume will not be removed. To remove the volume:
    ```sh
    docker volume ls
    docker volume rm "mysql_volume_name"
    ```

- To import database dumps automatically:
    - The MySQL data volume should not exist:
        - You are executing Docker Compose for the first time.
        - You removed the MySQL data volume.
    - Place your .sh, .sql and .sql.gz files on the host data/mysql/import host directory.
    - Run the containers.

- MySQL and Apache/PHP containers are different hosts.
    - To connect from Apache/PHP to MySQL you should replace "localhost" to "db" (the MySQL hostname in the Docker network) 

## Known limitations

Apache process user will be created and assigned at build time. Each time you need to change the user/uid and group/gid you must re-build the image.

## Support

Please [open an issue](https://github.com/fmfpereira/dlamp/issues/new) for support.

## Contributing

Please contribute using [Github Flow](https://guides.github.com/introduction/flow/). Create a branch, add commits, and [open a pull request](https://github.com/fmfpereira/dlamp/compare).
