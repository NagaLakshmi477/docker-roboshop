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


## Docker Compose Commands

### Running a Single Container

Without Docker Compose, we start each container individually.

Example:

```bash
docker run -d --name mongodb --network roboshop lakshmi1092/mongodb:v1
```

---

### Running All Services

With Docker Compose, all services can be started using a single command:

```bash
docker compose up -d
```

**Options Used:**

- `-d` → Runs all services in detached mode (background).

---

## Verify Running Services

```bash
docker ps
```

This command displays all running containers.

---

## Recreate Services

Whenever we make changes to the application files or the Compose configuration, run:

```bash
docker compose up -d
```

Docker Compose recreates the services if required and applies the latest changes.

# Payment

## Build Docker Image

```bash
docker build -t lakshmi1092/payment:v1 .
```

> **Note:** Environment variables can be provided either at **build time** or **run time**, depending on the application requirement.

---

## Start the Services

```bash
docker compose up -d
```

This command starts (or recreates) the services using the latest configuration.

---

# Frontend

## Build Docker Image

```bash
docker build -t lakshmi1092/frontend:v1 .
```

---

## Start the Services

```bash
docker compose up -d
```

This command starts (or recreates) the services using the latest configuration.

# Application Troubleshooting

## Issue 1: Categories Not Loading

### Login to the Frontend Container

```bash
docker exec -it frontend bash
```

Go to the Nginx HTML directory:

```bash
cd /usr/share/nginx/html
```

List the files:

```bash
ls -l
```

---

## Update the Dockerfile

Copy the static files into the Nginx HTML directory.

```dockerfile
ADD static /usr/share/nginx/html/
```

Rebuild the image after updating the Dockerfile.

---

## Still Facing the Issue?

Check the Nginx configuration.

Go to the Nginx configuration directory:

```bash
cd /etc/nginx
```

List the files:

```bash
ls -l
```

There is a directory named **conf.d**.

Go inside it:

```bash
cd /etc/nginx/conf.d
```

List the files:

```bash
ls -l
```

> **Note:** The configuration files inside `conf.d` may override the settings in `nginx.conf`. Remove the unnecessary configuration if required.

---

## Rebuild the Frontend Image

Pull the latest changes:

```bash
git pull
```

Build the image without using the cache:

```bash
docker build -t lakshmi1092/frontend:v1 --no-cache .
```

Start the services again:

```bash
docker compose up -d
```

---

# Issue 2: Cities Data Not Loading

Login to the MySQL container:

```bash
docker exec -it mysql bash
```

Login to MySQL:

```bash
mysql -u root -pRoboShop@1
```

Show all databases:

```sql
show databases;
```

Select the **cities** database:

```sql
use cities;
```

Show the tables:

```sql
show tables;
```

Check whether the data is available:

```sql
select count(*) from cities;
```

---

## Root Cause

The initialization script is executed again, which drops and recreates the schema every time.

As a result, the existing data is deleted and recreated.

### Solution

Delete the unnecessary schema file and rebuild the MySQL image.

# Interview Scenario

If asked in an interview about a real-world database issue, you can explain it like this:

- Developers provide the `.sql` files to create the database schema and load initial data.
- During development, if there are changes to the database structure, developers update these SQL scripts.
- By mistake, two developers created different SQL files for the same database.
- One SQL file:
  - Created the table.
  - Inserted the required data.
- Another SQL file:
  - Dropped the table.
  - Recreated the table.
  - Did **not** insert the data again.
- As a result, all the data was lost.
- This happened in the **development environment**, but it became a major issue and was escalated because the application was unable to retrieve the required data.

---

# Containers are Ephemeral

By default, Docker containers are **ephemeral**.

This means:

- If a container is removed, all the data stored inside the container is also removed.

---

## Stop and Remove Containers

```bash
docker compose down
```

This command stops and removes all the containers created by Docker Compose.

---

## What Happens to the Data?

Suppose a user is already registered in the application.

After running:

```bash
docker compose down
```

and starting the application again,

the previous user's data is **not available** if it was stored only inside the container.

This is because containers are **ephemeral** by default.

When a container is deleted, its filesystem and data are also deleted.

---

## Next Step

To avoid losing data, we need to store it **outside the container** using **Docker Volumes (persistent storage)** instead of storing it only inside the container's filesystem.


# Docker Volumes

## Why Do We Need Volumes?

By default, containers are **ephemeral**.

If a container is removed, all the data stored inside the container is also removed.

To persist data even after the container is deleted, we use **Docker Volumes (or bind mounts).**

---

## Example Without a Volume

Run an Nginx container:

```bash
docker run -d -p 8080:80 nginx
```

Login to the container:

```bash
docker exec -it <container_id> bash
```

Go to the HTML directory:

```bash
cd /usr/share/nginx/html
```

Create a test file:

```bash
echo "hello" > hello.html
```

Exit the container:

```bash
exit
```

Remove the container:

```bash
docker rm -f <container_id>
```

If you create a new Nginx container again, the `hello.html` file will **not** be present because the previous container was deleted.

---

## Where is the Data Stored?

Become the root user:

```bash
sudo su -
```

Inspect the container:

```bash
docker inspect <container_id>
```

Docker creates a storage directory on the host and stores the container's filesystem there.

Example:

```text
/var/lib/docker/overlay2/...
```

The merged filesystem is available under a path similar to:

```text
.../merged/usr/share/nginx/html/
```

> **Note:** Docker manages these directories automatically. They should not be modified manually.

---

# Using a Volume (Bind Mount)

Create a directory on the host machine:

```bash
mkdir nginx-data
```

Run the Nginx container with a bind mount:

```bash
docker run -d -p 8080:80 \
-v /home/ec2-user/nginx-data:/usr/share/nginx/html \
nginx
```

### Explanation

- `/home/ec2-user/nginx-data` → Directory on the host machine.
- `/usr/share/nginx/html` → Directory inside the container.

Any files created inside `/usr/share/nginx/html` are stored in the host directory `/home/ec2-user/nginx-data`.

Even if the container is removed, the data remains on the host.

> **Note:** This example uses a **bind mount**, where a host directory is mapped to a directory inside the container. Docker also supports **named volumes** for persistent storage.
# Image Optimization

## Build and Push All Images

Login to Docker Hub:

```bash
docker login -u lakshmi1092
```

Go to the project directory:

```bash
cd roboshop-docker
```

Build and push all images:

```bash
for i in $(ls -d */)
do
    cd "$i"
    name=$(basename "$i")
    docker build -t lakshmi1092/$name:v1 .
    docker push lakshmi1092/$name:v1
    cd ..
done
```

### Explanation

- `ls -d */` → Lists all directories.
- `cd "$i"` → Moves into each application directory.
- `basename "$i"` → Removes the trailing `/` and returns only the directory name.
- `docker build -t lakshmi1092/$name:v1 .` → Builds the Docker image.
- `docker push lakshmi1092/$name:v1` → Pushes the image to Docker Hub.
- `cd ..` → Returns to the parent directory.

---

## Image Optimization Techniques

### 1. Use Minimal Official Images

Use lightweight base images whenever possible to reduce the image size.

Example:

```dockerfile
FROM node:20-alpine3.21
```

Using Alpine-based images:

- Reduces image size.
- Improves image download and upload speed.
- Reduces storage usage.

---

## Bind Mount Syntax

```text
-v <host-directory>:<container-directory>
```

Example:

```bash
-v /home/ec2-user/nginx-data:/usr/share/nginx/html
```

- **Host Directory:** `/home/ec2-user/nginx-data`
- **Container Directory:** `/usr/share/nginx/html` (Nginx HTML directory)
  

# Docker Volumes

## Unnamed (Unmanaged) Volumes / Bind Mounts

- If **we create and manage the directory** on the host machine, it is called an **unnamed (unmanaged) volume** (commonly known as a **bind mount**).
- We are responsible for creating, managing, backing up, and deleting the directory.

**Example:**

```bash
docker run -d -p 8080:80 \
-v /home/ec2-user/nginx-data:/usr/share/nginx/html \
nginx
```

Here:

- `/home/ec2-user/nginx-data` → Created and managed by the user.
- `/usr/share/nginx/html` → Directory inside the container.

---

## Named (Managed) Volumes

- If **Docker creates and manages the storage**, it is called a **named (managed) volume**.
- Docker automatically creates the volume and stores the data.
- Docker manages the location of the volume on the host.

**Example:**

```bash
docker volume create nginx-data

docker run -d -p 8080:80 \
-v nginx-data:/usr/share/nginx/html \
nginx
```

Here:

- `nginx-data` → Docker-managed volume.
- `/usr/share/nginx/html` → Directory inside the container.
