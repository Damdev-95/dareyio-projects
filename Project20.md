# MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER &AMP; DOCKER COMPOSE

Instructions On How To Submit Your Work For Review And Feedback
To submit your work for review and feedback – follow this instruction.

Install Docker and prepare for migration to the Cloud
First, we need to install Docker Engine, which is a client-server application that contains:

A server with a long-running daemon process dockerd.
APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
A command-line interface (CLI) client docker.
You can learn how to install Docker Engine on your PC here

Before we proceed further, let us understand why we even need to move from VM to Docker.

As you have already learned – unlike a VM, Docker allocated not the whole guest OS for your application, but only isolated minimal part of it – this isolated container has all that your application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. It is a well-known problem when a developer sends his application to you, you try to deploy it, deployment fails, and the developer replies, "- It works on my machine!". With Docker – if the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

As a part of this project, you will use already well-known by you Jenkins for Continous Integration (CI). So, when it is time to write Jenkinsfile, update your Terraform code to spin up an EC2 instance for Jenkins and run Ansible to install & configure it.

To begin our migration project from VM based workload, we need to implement a Proof of Concept (POC). In many cases, it is good to start with a small-scale project with minimal functionality to prove that technology can fulfill specific requirements. So, this project will be a precursor before you can move on to deploy enterprise-grade microservice solutions with Docker.

You can start with your own workstation or spin up an EC2 instance to install Docker engine that will host your Docker containers.

Remember our Tooling website? It is a PHP-based web solution backed by a MySQL database – all technologies you are already familiar with and which you shall be comfortable using by now.

So, let us migrate the Tooling Web Application from a VM-based solution into a containerized one.

MySQL in container
Let us start assembling our application from the Database layer – we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

# Step 1: Pull MySQL Docker Image from Docker Hub Registry
* Start by pulling the appropriate Docker image for MySQL. You can download a specific version or opt for the latest release, as seen in the following command:

`docker pull mysql/mysql-server:latest`
* List the images to check that you have downloaded them successfully:
`docker image ls`
![image](https://user-images.githubusercontent.com/71001536/175141484-b5d8e1cc-991e-411e-ba9a-c93d8b6007e8.png)

# Step 2: Deploy the MySQL Container to your Docker Engine.
* Once you have the image, move on to deploying a new MySQL container with:
`docker run --name projectdb -e MYSQL_ROOT_PASSWORD=damilare -d mysql/mysql-server:latest`

![image](https://user-images.githubusercontent.com/71001536/175240323-79ae3831-40da-4183-8885-8501fab3be80.png)

* Then, check to see if the MySQL container is running: Assuming the container name specified is mysql-server
`docker ps -a`

# CONNECTING TO THE MYSQL DOCKER CONTAINER

* Approach 1

Connecting directly to the container running the MySQL server:

`$ docker exec -it mysql bash`

![image](https://user-images.githubusercontent.com/71001536/175242666-b5408a76-cc51-477f-8125-d4f37df598aa.png)

or
`$ docker exec -it mysql mysql -uroot -p`

![image](https://user-images.githubusercontent.com/71001536/175243145-9a648a69-d83b-4492-a311-49fb8f2b8c42.png)


Provide the root password when prompted. With that, you’ve connected the MySQL client to the server.

Finally, change the server root password to protect your database. Exit the the shell with exit command

Flags used

exec used to execute a command from bash itself
-it makes the execution interactive and allocate a pseudo-TTY
bash this is a unix shell and its used as an entry-point to interact with our container
mysql The second mysql in the command "docker exec -it mysql mysql -uroot -p" serves as the entry point to interact with mysql container just like bash or sh
-u mysql username
-p mysql password

* Approach 2

At this stage you are now able to create a docker container but we will need to add a network. So, stop and remove the previous mysql docker container.
```
docker ps -a
docker stop mysql 
docker rm mysql or <container ID> 04a34f46fb98
```
verify that the container is deleted
![image](https://user-images.githubusercontent.com/71001536/175243851-0538412c-eb84-4b12-a31d-15c5b16ae2ef.png)

* First, create a network:

`$ docker network create --subnet=192.168.10.0/24 tooling_app_network`

* Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of DRIVER Bridge. So, also, it is the default network. You can verify this by running the docker network ls command.

But there are use cases where this is necessary. For example, if there is a requirement to control the cidr range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the --subnet

![image](https://user-images.githubusercontent.com/71001536/175245037-dd586049-ffbc-4907-91a0-a87bd6f7cc16.png)

* For clarity’s sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.
 Run the MySQL Server container using the created network.

* First, let us create an environment variable to store the root password:

 `$ export MYSQL_PW=`
verify the environment variable is created

* Then, pull the image and run the container, all in one command like below:

```
$ docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest 
```
## Flags used

* -d runs the container in detached mode
* --network connects a container to a network
* -h specifies a hostname

![image](https://user-images.githubusercontent.com/71001536/175246484-ab29c95c-b110-4cd3-982e-79ddf6c96952.png)

* Create a file and name it create_user.sql and add the below code in the file:

` $ CREATE USER 'admin'@'%' IDENTIFIED BY 'project20'; GRANT ALL PRIVILEGES ON * . * TO 'admin'@'%';` 

* Run the script:
Ensure you are in the directory create_user.sql file is located or declare a path

 `$ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql` 
 
 ![image](https://user-images.githubusercontent.com/71001536/175249097-7bb60e6d-8a3f-4b19-8f9b-67d87228218f.png)

Run the MySQL Client Container:

` $ docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -uadmin  -p ` 

## Flags used:

* --name gives the container a name
* -it runs in interactive mode and Allocate a pseudo-TTY
* --rm automatically removes the container when it exits
* --network connects a container to a network
* -h a MySQL flag specifying the MySQL server Container hostname
* -u user created from the SQL script
* admin username-for-user-created-from-the-SQL-script-create_user.sql
* -p password specified for the user created from the SQL script

![image](https://user-images.githubusercontent.com/71001536/175252868-b6003662-23eb-4153-bb69-614a71f325ce.png)

# Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

* Clone the Tooling-app repository from here

` $ git clone https://github.com/darey-devops/tooling.git `

* On your terminal, export the location of the SQL file
` $ export tooling_db_schema=/html/tooling_db_schema.sql `

* You can find the tooling_db_schema.sql in the tooling/html/tooling_db_schema.sql folder of cloned repo.
Verify that the path is exported

`echo $tooling_db_schema`

![image](https://user-images.githubusercontent.com/71001536/175257810-3cd5f8a2-cb65-4914-b4fc-d085d08633a4.png)

* Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.
` $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema` 

![image](https://user-images.githubusercontent.com/71001536/175262113-0821d0d8-fecd-4c7b-a5ef-f1245597a7df.png)

* Update the .env file with connection details to the database

The .env file is located in the html tooling/html/.env folder but not visible in terminal. you can use vi or nano
```
sudo vi .env

MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
Flags used:

MYSQL_IP mysql ip address "leave as mysqlserverhost"
MYSQL_USER mysql username for user export as environment variable
MYSQL_PASS mysql password for the user exported as environment varaible
MYSQL_DBNAME mysql databse name "toolingdb"
```
## Run the Tooling App

Containerization of an application starts with creation of a file with a special name - 'Dockerfile' (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, you will build your container from a pre-created Dockerfile, but as a DevOps, you must also be able to write Dockerfiles.

So, let us containerize our Tooling application; here is the plan:

* Make sure you have checked out your Tooling repo to your machine with Docker engine
First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.

* Launch the container with docker run
`docker run --network tooling_app_network -p 8090:80 -it phptodo:0.0.1 `
* Try to access your application via port exposed from a container

* Ensure you are inside the directory "tooling" that has the file Dockerfile and build your container :

 `$ docker build -t tooling:0.0.1 . `

In the above command, we specify a parameter -t, so that the image can be tagged tooling"0.0.1 - Also, you have to notice the . at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.
![image](https://user-images.githubusercontent.com/71001536/175271030-75ac1bac-3d51-4085-8366-16bc64516751.png)

* Run the container:
` $ docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1` 

## Let us observe those flags in the command.

```
We need to specify the --network flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
The -p flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.
```
![image](https://user-images.githubusercontent.com/71001536/175274577-829326ce-9590-4945-a0ba-0fc5e810b0b7.png)

![image](https://user-images.githubusercontent.com/71001536/175293806-9bd4cf58-a9e9-4555-a7ae-630d0503a6b8.png)

 ```
 GRANT ALL ON * . * TO 'admin'@'%';
 FLUSH PRIVILEGES;
 ```
 ## I encounter issues at this point(SINCE YESTERDAY)
 
 * I later discvered that my sql-server is on the default network not the custom network 
 * I really learnt alot, troubleshooting the docker, mysql though it takes my time 
 
 ![image](https://user-images.githubusercontent.com/71001536/175499719-190a0b99-3d9b-415e-bf79-2d9c674ce63c.png)
 
 ![image](https://user-images.githubusercontent.com/71001536/175500185-96d9dc1a-7c10-4102-be79-575c0bbc8cd9.png)


# Practice Task №1 – Implement a POC to migrate the PHP-Todo app into a containerized application.

The project below will challenge you a little bit, but the experience there is very valuable for future projects.

# Part 1

* Write a Dockerfile for the TODO app
## Dockerfile
```
FROM php:7.4.30-cli

USER root
WORKDIR  /var/www/html

RUN apt-get update && apt-get install -y \
    libpng-dev \
    zlib1g-dev \
    libxml2-dev \
    libzip-dev \
    libonig-dev \
    zip \
    curl \
    unzip \
    && docker-php-ext-configure gd \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install zip \
    && docker-php-source delete

COPY . .

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

ENTRYPOINT [  "bash", "start-apache.sh" ]
```

Run both database and app on your laptop Docker Engine
` docker build -t phptodo:0.0.1 . `
![image](https://user-images.githubusercontent.com/71001536/175507593-54885d04-65e5-4a6c-8e6d-f1eeaecce984.png)

# Laravel default port is 8000

`docker run --network tooling_app_network --name=php-todo -p 8090:8000 -it phptodo:0.0.1 ` 
Access the application from the browser

![image](https://user-images.githubusercontent.com/71001536/175517718-2abf1758-7cd9-4e7a-a7d2-3e2ace5419b9.png)

# Part 2
* Create an account in Docker Hub
* Create a new Docker Hub repository
* Push the docker images from your PC to the repository

Part 3
Write a Jenkinsfile that will simulate a Docker Build and a Docker Push to the registry
Connect your repo to Jenkins
Create a multi-branch pipeline
Simulate a CI pipeline from a feature and master branch using previously created Jenkinsfile
Ensure that the tagged images from your Jenkinsfile have a prefix that suggests which branch the image was pushed from. For example, feature-0.0.1.
Verify that the images pushed from the CI can be found at the registry.
Deployment with Docker Compose
All we have done until now required quite a lot of effort to create an image and launch an application inside it. We should not have to always run Docker commands on the terminal to get our applications up and running. There are solutions that make it easy to write declarative code in YAML, and get all the applications and dependencies up and running with minimal effort by launching a single command.

In this section, we will refactor the Tooling app POC so that we can leverage the power of Docker Compose.

First, install Docker Compose on your workstation from here
Create a file, name it tooling.yaml
```
Begin to write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
```
The YAML file has declarative fields, and it is vital to understand what they are used for.
version: Is used to specify the version of Docker Compose API that the Docker Compose engine will connect to. This field is optional from docker compose version v1.27.0. You can verify your installed version with:
`docker-compose --version`
docker-compose version 1.28.5, build c4eb3a1f
service: A service definition contains a configuration that is applied to each container started for that service. In the snippet above, the only service listed there is tooling_frontend. So, every other field under the tooling_frontend service will execute some commands that relate only to that service. Therefore, all the below-listed fields relate to the tooling_frontend service.
build
port
volumes
links
You can visit the site here to find all the fields and read about each one that currently matters to you -> https://www.balena.io/docs/reference/supervisor/docker-compose/

You may also go directly to the official documentation site to read about each field here -> https://docs.docker.com/compose/compose-file/compose-file-v3/

Let us fill up the entire file and test our application:
```
version: "3.9"
services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: <The database name required by Tooling app >
      MYSQL_USER: <The user required by Tooling app >
      MYSQL_PASSWORD: <The password required by Tooling app >
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
volumes:
  tooling_frontend:
  db:
  ```
* Run the command to start the containers
`docker-compose -f tooling.yaml  up -d` 

 
 * Verify that the compose is in the running status:
`docker compose ls`

 ## Practice Task №2 – Complete Continous Integration With A Test Stage
* Document your understanding of all the fields specified in the Docker Compose file tooling.yaml
* Update your Jenkinsfile with a test stage before pushing the image to the registry.
* What you will be testing here is to ensure that the tooling site http endpoint is able to return status code 200. Any other code will be determined a stage failure.
* Implement a similar pipeline for the PHP-todo app.
* Ensure that both pipelines have a clean-up stage where all the images are deleted on the Jenkins server.
