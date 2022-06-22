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

# Step 2: Deploy the MySQL Container to your Docker Engine. Once you have the image, move on to deploying a new MySQL container with:
