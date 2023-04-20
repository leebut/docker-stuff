# Docker Compose

This is a working document from my own experience using Docker. If you see any errors or improvements, please do comment.

Docker Compose is great because you can link several containers in one docker-compose file that takes care of pulling, buliding and running all of containers from one command: `sudo docker-compose up` or `sudo docker-compose up -d` (for background running)

Docker compose can be configured to not have to type in *sudo* every time, but please read the docs for that.

I use Docker (**version: 2.17.2**) on Manjaro Linux, which is a fork of Arch. I recently installed Docker (**version: 1.29.2**) on Ubuntu , running on a virtual machine as well for testing.

On my Manjaro box I can use the command `docker compose up`, but on Ubuntu, I have to use `sudo docker-compose up`. I'm just creating awareness that systems, versions and setups differ.

So, it appears that if you have < version 2, a hyphen in `docker-compose` is required.

## Installation

You can install Docker Compose along with Docker, but if you already have Docker, just install Docker Compose.

You can use a GUI software manager or a terminal.

**Debian**

`sudo apt install docker docker-compose`

**Arch**

`sudo pacman -S docker docker-compose`

---

## Set Up a Project

### docker-compose.yaml

Docker Compose searches for a file named **docker-compose.yaml**, which is where our docker image configuration instructions are. Some images are built from a **`Dockerfile`** as well, but we don't need one for MySQL. We will look more into docker-compose.yaml later, but first let's get the directories sorted out.

1) create a new project directory: `mkdir directory_name`

2) Change to the directory: `cd directory_name`

3) Create a new file: `touch docker-compose.yaml`

- NOTE: you could open an empty file with a text editor:

`nano docker-compose.yaml`

4) Here is a docker-compose.yaml similar to the file I used to create my MySQL container named **my_database** which creates a database named **pies**.

```
version: "3"

services:
  data:
    image: mysql
    container_name: my_database
    environment:
      MYSQL_DATABASE: pies
      MYSQL_ROOT_PASSWORD: 'gren5432ko'
      MYSQL_USER: norooty
      MYSQL_PASSWORD: 'gkke532lro'
    volumes:
      - ./mysql:/var/lib/mysql
    ports:
      - 3306:3306

```

### What does the file do?

**Version** and **services** are required.

- **data**: this the name you give to the service. It could be named db, mysql, whatever_you_want.  
    + The label is used when linking to other containers in the image.

- **image**: instructs Docker Compose to pull (download) MySQL from [Docker Hub](https://hub.docker.com/_/mysql).

- **container_name**: gives the Docker container a custom name, which will be used later.

- **environment**: sets up the database and in this case:

+ MYSQL_DATABSAE: Creates a new database during the build process

+ MYSQL_ROOT_PASSWORD: sets a root password for the database

+ MYSQL_USER: creates a non-root user

+ MYSQL_PASSWORS: sets the non-root user's password

- **volumes**: maps a local directory in our project folder (or any other directory you define) to where MySQL works from, a bit like syncing. This persists the data locally as well as in the container, so if we remove the container, we can rebuild it, and the data will populate the database from the local directory during the build process.

+ Syntax: `local_directory:container_directory`

+ The `./mysql` part creates a new directory in the project directory `:/var/lib/mysql` is were any data will be *synched* from.

+ The name of the local directory can be anything that is meaningful.

- **ports**: MySQL's default port is 3306. To reach it from our host, we need to map it to port 3306 (or any other port) on our machine.

+ `local_port:container_port`

## Building the Image

To build the docker image, depending on your set up: `sudo docker-compose up` or `sudo docker compose up`

You can use `sudo docker-compose up` to start docker with rebuild if changes are detected in the docker-compose.yaml file. To only start the container, type, `sudo docker-compose start`.

Docker Compose will build and start the container. We can check the container is running with: `sudo docker ps`.

In the last column, you should see the container name.

```

NAMES

my_database

```

## Accessing MySQL

You can set up MySQL Workbench as you would in any case, but set the host to **127.0.0.1** NOT localhost.

I will use the information from the **docker-compose.yaml** configuration **above**.

To use Docker in a terminal, we need to command Docker to execute MySQL:

`sudo docker exec -it my_database mysql -u norooty -p`

I'm sure you don't want to do that every time, so let's create a BASH script. I'm not sure about ZSH scripts.

In the project folder, create a new file with a short name. I named mine, `go_sql.sh`.

Open the file with these contents (change according to your configuration):

```

#!/bin/bash

sudo docker exec -it my_database mysql -u norooty -p

```

- **exec**: tells Docker to execute the command

- **-it**: makes the shell interactive

- **my_database**: the container name

- **mysql**: what we want Docker to run with the container

- - **-u**: mysql user name

- - **norooty**: the non-root username

- - **-p**: This produces a password prompt in the terminal

- Save and exit the file.

- Set the file permissions to 771: `chmod 771 go_mysql.sh` If anyone knows more about that, please comment.

Now, you can start a MySQL shell, while in the project directory with `./go_mysql.sh`.

## Stopping the Container

To stop a container if you started it with docker-compose up, you will see activity output to the terminal. Press **CTRL** + **C**.

If you started it with `sudo docker-compose up -d`, there is no logs output. Type, `sudo docker-compose stop`

## Removing a Container

To remove a container, not including persistent data in the local directory, type, `sudo docker-compose down`

## Start a Container

You can use `sudo docker-compose up` or `sudo docker-compose up -d`

up will check for changes in the docker-compose.yaml file and recreate the container if needed.

**-d** runs Docker in the background.

Docker may be started without checking for changes, with `sudo docker-compose start`

---

That should cover everything to get a Docker Compose image up and running with one container.

Docker images are found on [Docker Hub](https://hub.docker.com/_/mysql).