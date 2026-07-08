# Roboshop Docker

# MongoDB

## Overview

- We are using **MongoDB 7** official image from Docker Hub.
- Since it is an official image, there is **no need to install or manually start MongoDB** inside the container.
- The image already contains everything required to run the MongoDB server.

---

## Requirement

The **Catalogue** service requires some default data in MongoDB.

To load this data automatically, we use MongoDB's initialization feature.

---

## MongoDB Initialization

When a MongoDB container starts **for the first time**, it checks the following directory:

```text
/docker-entrypoint-initdb.d
```

If this directory contains any:

- `.sh` files
- `.js` files

MongoDB automatically executes them.

### Notes

- These files run **only during the first container startup**.
- Files are executed in **alphabetical order**.
- This is the recommended way to create databases, collections, users, and load initial data.

---

## Build Docker Image

```bash
docker build -t mongodb:v1 .
```

---

## List Images

```bash
docker images
```

---

## Run MongoDB Container

```bash
docker run -d --name mongodb mongodb:v1
```

### Options Used

- `-d` → Run the container in detached mode (background).
- `--name mongodb` → Assign the container name as **mongodb**.

> **Note:** We did **not** use the `-p` option because MongoDB is used only by other Docker containers inside the Docker network. It does not need to be exposed to the outside host.

---

## Verify Running Containers

```bash
docker ps
```

Example Output:

```text
CONTAINER ID   IMAGE         NAME
xxxxxxxxxxxx   mongodb:v1    mongodb
```

This confirms that the MongoDB container is running successfully.

---


# Catalogue

## Overview

- For the **Catalogue** service, we use the official **Node.js** image as the base image.
- We can place our customized application files under the `/opt/server` folder inside the container.
- This folder is used to store the Catalogue application code.

---

## Build Docker Image

```bash
docker build -t catalogue:v1 .
```

---

## List Images

```bash
docker images
```

---

## Run Catalogue Container

```bash
docker run -d --name catalogue catalogue:v1
```

### Options Used

- `-d` → Run the container in detached mode (background).
- `--name catalogue` → Assign the container name as **catalogue**.

---

## Verify Running Containers

```bash
docker ps
```

This confirms that the Catalogue container is running.

---

## Access the Container

```bash
docker exec -it catalogue bash
```

### Options Used

- `docker exec` → Execute a command inside a running container.
- `-i` → Keep STDIN open (interactive mode).
- `-t` → Allocate a terminal.
- `bash` → Open a Bash shell inside the container.

---

## Health Check

To verify that the Catalogue service is running properly:

```bash
curl http://localhost:8080/health
```

If the service is running successfully, you should receive a successful response, which indicates that the Catalogue application is healthy.


# Failure

Docker is unable to connect to MongoDB.

Possible errors:

- **Instana agent not reachable** → No metrics/tracing.
- **MongoDB connection error** → Application cannot connect to its database.

---

## Check Container Logs

```bash
docker logs catalogue
```

---

## Check Network Interfaces

```bash
ifconfig
```

### Note

- Who provides the internet to the VM?
  - **AWS** provides the internet through the **Ethernet interface**.
- When Docker is installed, it internally (virtually) creates one network.
- This network is called **docker0** (Docker Bridge Network).

---

## Docker Bridge Network

Whenever we create a Docker container, Docker automatically allocates an IP address.

Example:

```text
docker0 (Gateway)      : 172.17.0.1
Catalogue Container    : 172.17.0.2
MongoDB Container      : 172.17.0.3
```

To check the Catalogue container IP:

```bash
docker inspect catalogue
```

To check the MongoDB container IP:

```bash
docker inspect mongodb
```

---

## Problem

Both containers are in the **same Docker network**, but they are still unable to communicate because Docker is using the **default bridge network**.

---

The default bridge network allocates IP addresses to containers, but it does not automatically map container names to those IP addresses.

## Docker Network Commands

To view Docker networks:

## Docker Networks

To list all Docker networks:

```bash
docker network ls
```

This command displays the list of available Docker networks.

---

## Types of Docker Networks

### Bridge Network

- Docker creates a **bridge network** by default.
- Whenever Docker is installed on a server, it automatically creates a bridge called **docker0**.
- `docker0` is the bridge that allocates IP addresses to Docker containers.
- Docker creates a separate virtual network interface and assigns IP addresses to containers.

> Bridge network means Docker creates a separate virtual network and assigns IP addresses to containers.

---

### Host Network

- In the **host network**, the container directly uses the host machine's network.
- The host machine provides the network connectivity to the container.
- The container does not get a separate Docker bridge network IP.

> Host network means the container is directly connected to the host network.

---

## Why Create a Custom Bridge Network?

- By default, Docker creates the **bridge network** whenever Docker is installed.
- We create a separate (custom) bridge network because the **default bridge network does not automatically resolve container names**.
- Containers on the default bridge network can communicate using **IP addresses**, but not automatically using **container names**.
- Docker always recommends creating a **custom bridge network** so containers can communicate using container names (DNS).



# Create a Custom Bridge Network

Create a custom Docker network:

```bash
docker network create roboshop
```

---

## Disconnect Containers from the Default Bridge Network

Before connecting the containers to the custom network, disconnect them from the default **bridge** network.

```bash
docker network disconnect bridge catalogue
```

```bash
docker network disconnect bridge mongodb
```

---

## List Docker Networks

```bash
docker network ls
```

---

## Connect Containers to the Custom Network

Connect the MongoDB container:

```bash
docker network connect roboshop mongodb
```

Connect the Catalogue container:

```bash
docker network connect roboshop catalogue
```

---

## Verify the Network Configuration

Check the MongoDB container:

```bash
docker inspect mongodb
```

Check the Catalogue container:

```bash
docker inspect catalogue
```

Verify that both containers are connected to the **roboshop** network.

---

## Verify the Application

Access the Catalogue container:

```bash
docker exec -it catalogue bash
```

Run the health check:

```bash
curl localhost:8080/health
```

If the health check is successful, the Catalogue container is able to communicate with the MongoDB container.

> **Note:** Here we are using a **custom bridge network** (`roboshop`) so that containers can communicate using their container names.

# Redis

## Overview

- Here we are **not creating a custom image**.
- We are only running the Redis server; no default data needs to be loaded.
- Therefore, we can directly use the official Redis image from Docker Hub.

---

## Run Redis Container

```bash
docker run -d --name redis --network roboshop redis:7
```

> **Note:** If the Redis image is not available locally, Docker automatically pulls it from Docker Hub.

---

## Verify the Container

```bash
docker ps
```

The Redis container should be in the **running** state.

---

# User

## Build Docker Image

```bash
docker build -t user:v1 .
```

---

## Run User Container

```bash
docker run -d --name user --network roboshop user:v1
```

---

## Verify the Container

```bash
docker ps
```

---

## Access the Container

```bash
docker exec -it user bash
```

---

## Health Check

```bash
curl localhost:8080/health
```

---

# Cart

## Build Docker Image

```bash
docker build -t cart:v1 .
```

---

## Run Cart Container

```bash
docker run -d --name cart --network roboshop cart:v1
```

---

## Verify the Container

```bash
docker ps
```

---

## Access the Container

```bash
docker exec -it cart bash
```

---

## Health Check

```bash
curl localhost:8080/health
```

---

# MySQL

## Build Docker Image

```bash
docker build -t mysql:v1 .
```

---

## Run MySQL Container

```bash
docker run -d --name mysql --network roboshop mysql:v1
```

---

## Verify the Container

```bash
docker ps
```

---

## Access the Container

```bash
docker exec -it mysql bash
```

---

# Shipping

## Build Docker Image

```bash
docker build -t shipping:v1 .
```

> **Note:** Don't forget the `.` at the end of the `docker build` command.

---

## Run Shipping Container

```bash
docker run -d --name shipping --network roboshop shipping:v1
```

---

## Verify the Container

```bash
docker ps
```

---

## Access the Container

```bash
docker exec -it shipping bash
```

---

## Health Check

```bash
curl localhost:8080
```
# Push Docker Images to Docker Hub

## Login to Docker Hub

```bash
docker login -u lakshmi1092
```

After running the command, enter your Docker Hub password when prompted.

---

## Build and Push Images

```bash
for i in cart catalogue mongodb mysql shipping user
do
    cd $i
    docker build -t lakshmi1092/$i:v1 .
    docker push lakshmi1092/$i:v1
    cd ..
done
```

### Explanation

- `for i in ...` → Loops through each application directory.
- `cd $i` → Moves into the application's directory.
- `docker build -t lakshmi1092/$i:v1 .` → Builds the Docker image and tags it.
- `docker push lakshmi1092/$i:v1` → Pushes the image to your Docker Hub repository.
- `cd ..` → Returns to the parent directory and continues with the next application.

--- 

# Docker Compose

Before using Docker Compose, build and push all application images to Docker Hub.

---

## Login to Docker Hub

```bash
docker login -u lakshmi1092
```

Enter your Docker Hub password when prompted.

---

## Build and Push All Images

```bash
for i in cart catalogue mongodb mysql shipping user
do
    cd $i
    docker build -t lakshmi1092/$i:v1 .
    docker push lakshmi1092/$i:v1
    cd ..
done
```

> **Note:** After all images are available in Docker Hub, Docker Compose can pull these images and start all the services together using a single `docker-compose.yml` or `compose.yaml` file.

# Docker Compose

Docker Compose is a **command-line tool** used to manage **multi-container applications**.

---

## Features

- We can define all Docker containers as **services** in a single Compose file.
- We can create dependencies between services.
- Start all services at once.
- Stop all services at once.
- Rebuild services when changes are made.
- View the status of running services.
- Stream the log output of running services.

here catalogue depends on momgodb
cart depends on catalogue 

## Why Docker Compose?

In a multi-container application, services depend on each other.

For example:

- Cart depends on Catalogue.
- Catalogue depends on MongoDB.
- Shipping depends on MySQL.

If we start the **Cart** service without starting **Catalogue** and **MongoDB**, the application will not work properly.

Without Docker Compose, we need to remember the order in which services should be started and run each container manually.

To avoid this, we use **Docker Compose**, which manages all the services together.


what is our run command:
-----------------------
docker run -d --name mongodb --network roboshop joindevops/mongodb:v1

docker compose up -d 

-d ---> sending to background

docker ps 

when ever we chang the files we need again run 
docker compose up -d 

----------------------
payment:

docker build -t lakshmi1092/payment:v1
# we can give environemnt variables on run time or build time
docker compose up -d 

--------------------
frontend:
docker build -t lakshmi1092/frontend:v1
docker compose up -d 

Application : break
======================
login into frontend:
docker exec -it frontend bash
cd /usr/share/nginx/html
ls -l
changes in docker file:
ADD static  /usr/share/nginx/html/
still same issues:
catagories no loaded
cd /etc/nginx/
ls -l
# there is a file conf.d this may over the nginx.conf data
cd /etc/nginx/conf.d
ls -l
so we need to remove that 
git pull
docker build -t lakshmi1092/frontend:v1 --no-cache .
docker compose up -d 
# now cities are not loadig
docer exec -it mysql:v1 bash
mysql -u root -pRoboShop@1
show databases
use citites
show tables
select count(*) from cities
# here in the scrpit it is creatinga nd updatng data gain it droping and creating
delete schema file
we can tell in inteview:
========================
devlopers provides the .sql files if there are any changes in db strcuture , by mistakley 2 develpoers created same sql filewith differnt names
one sql file is created the table and insrted the data. another file droping the table 
recreadted but does,'t insert the data . se we lost the data in db
this happened in dev environement but it becaome a big issue and escalated

container are ephemeral by deafult if you remove them it will remove data by default

to down-----> docker compose down
so here if we done the docer is that previous user is presentn or not present
no it is not present beacuse contanser are phemeral by default if we removethem they will remove entrie data by default
# now we need to optizise the decrasing image size and storing data in temporary
solution:
docker volumns
---------------
cd ..
docker run -d -p 8080:80 nginx
docker exec -it <id> bas    h
we can write one html file for testing
cd /usr/share/nginx/html/
echo "hello" > hello.html
exit
docker rm -f <id> ----remove and run gain we will see the data is present or not
where the data is present:
sudo su -
docker insepect <id>
it creates random dir and stroe the data into them
cd /merged/usr/share/nginx/html/
-----volums---------
mkdir nginx

docker run -d - 8080:80 -v /home/ec2-user/nginx-data:/usr/nginx/html nginx
it will stre the data in given floder

===========================================
Optimization:
===================
docker login -u lakshmi1092
cd roboshop-docker

for i in $(ls -d */);
do cd $i;
name=$(basename "$i);
docker build -i lakshmi1092/$name:v1 . ;
docker push lakshmi1092/$name 

this is build image and push images
base name means it removes the / in names it gives  only names

1. use minimal official images
catalogue --> FROM node:20-alphine3.21

-v host-dir:container-dir
/usr/share/nginx/html ----> nginx html dir

un-named/un-manged volumes:
===============================
it we create dir and manage it then those are called un-named/un-manged volumes
if docker creates and dir and manag then those are called named/ managed volumes

