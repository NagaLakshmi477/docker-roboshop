# roboshop-docker

ROBOSHOP 
==========
mongodb --7 version ---> official image from docker hub 
we will take it from docker hub. So no need to install and start the image
We need to add some default data regrading catalogue.
initializing a fresh instance:
There is default location
when a container is started 1st time it will execute files with .sh and .js that found in the location 
/docker/entrypoint-initdb.d 
files will be executed in alphabatical order
start,stop,and data loading ---> all will come from image
docker build -t mongodb:v1 .
docker images
docker run -d --name mongodb mongodb:v1
here 
-p ---> is not given bacuse we are not exposed outside
docker ps ----> mongodb is runnng
--------------------------------------------
docker build -t catalogue:v1 .
docker images
docker run -d --name catalogue catalogue:v1
docker ps
docker exec -it catalogue bash
curl http://localhost:8080/health  ---> sucess mesans catalogue runnng fine 
failure:
=======
docker logs catalogue
ifconfig 
who will provid the internet to vm ---> aws --> this is ethernet port 
here we are installing docker so docker inetrenally/viruvally created one network ---> this is called docker o ---> This is like mode M.
whenever we are creating docker container it will allocate the IP 
docker modem ---> (172.17.0.1)
docker inspect catalogue ---> It contain one IP (172.17.0.2)
docker inspect mongodb ---> It contain one IP (172.17.0.3)
Here those are in same network but unable to connect. Beacause docker uses default network
docker network docker netrwork ls ---> To see the list of networks

there are 2 types of networks:
bridge network
host network
Docker by default bridge network 
ex: docker 0 is the bridge that allocates Ip to components --> called bridge
host network:
modem gives the IP to the components. It will directly connected with host network
by deafult docker created bridge network whenever you installed docker inside any server
we can create sepaarte network beacuse both componets are not connected by default


host means directly host network.
bridge means docker create separate network interface and assign the IP address to conatiners.
docker default network can't comminucate between conatainers
docker always suggest to create custom bridge network


docker network create roboshop ----> to crate network
before connectining we need to disconnect from bridge:
docker network disconnect bridge catalogue
docker network disconnect bridge mongodb
docker network ls
docker network connect roboshop mongodb
docker network connect roboshop catalogue
docker inspect mongodb
docker inspect catalogue
docker exec -it catalogue bash
cutl localhost:8080/health 
# now it is connected
here we need to use customized network

-------------------
redis:
here we are installing redis, not filling the data. so we can run directly
docker run -d --name redis --network roboshop redis:7.
there is no local image so it can directly pull the image fom hub
docker ps ---It is running

--------------------
user:
docker build -t  user:v1 .
docker run -d --name user --network roboshop user:v1
docker ps
docker exec -it user
curl localhost 8080

-----------------------
cart:

docker build -t cart:v1
docker run -d --name cart --network roboshop cart:v1
docker ps
docker exec -it cart
curl localhost 8080

---------------------
mysql:

docker build -t mysql:v1
docker run -d --name mysql --network roboshop mysql:v1
docker ps
docker exec -it mysql

-----------------
shipping:

docker build -t shipping:v1
docker run -d --name shipping --network roboshop shipping:v1
docker ps
docker exec -it shipping
curl localhost 8080

-------------------------------next day
docker login -u lakshmi315
for i in cart catalogue mongodb mysql shipping user; do cd $i; docker build -t joindevops/$i:v1 . ; docker push lakshmi315/$i ; cd .. ; done    

docker compose:
================
docker login -u lakshmi315
for all images  run and building:
----------------------
for i in cart catalogue mongodb mysql shipping user ; do cd $i; docker build -t lakshmi315/$i:v1 . ; docker push lakshmi315/$i:v1 ; cd ..; done

it is a command line tool for  multi container application
we can define all docker containers as service. 
create dependies b/w them start them at onces and stop them at onces 
- start and stop, rubuild the services 
- view the status of running services
- stream the log output of running services

here catalogue depends on momgodb
cart depends on catalogue 

here we we run cart without catalogue and db the applications is not run. and we need to remebers the flow which we need to run 1st
so instead of we have docker compose


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

docker build -t lakshmi315/payment:v1
# we can give environemnt variables on run time or build time
docker compose up -d 

--------------------
frontend:
docker build -t lakshmi315/frontend:v1
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
docker build -t lakshmi315/frontend:v1 --no-cache .
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

to down-----> docker compose down
so here if we done the docer is that previous user is presentn or not present
no it is not present beacuse contanser are phemeral by default if we removethem they will remove entrie data by default
# now we need to optizise the decrasing image size and storing data in temporary
solution:
docker volumns
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
docker login -u lakshmi315
cd roboshop-docker

for i in $(ls -d */);
do cd $i;
name=$(basename "$i);
docker build -i lakshmi315/$name:v1 . ;
docker push lakshmi315/$name 

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

