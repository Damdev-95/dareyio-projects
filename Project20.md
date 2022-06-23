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
* Flags used

-d runs the container in detached mode
--network connects a container to a network
-h specifies a hostname

![image](https://user-images.githubusercontent.com/71001536/175246484-ab29c95c-b110-4cd3-982e-79ddf6c96952.png)

* Create a file and name it create_user.sql and add the below code in the file:

` $ CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%';` 

* Run the script:
Ensure you are in the directory create_user.sql file is located or declare a path

 `$ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql` 
 
 ![image](https://user-images.githubusercontent.com/71001536/175249097-7bb60e6d-8a3f-4b19-8f9b-67d87228218f.png)

