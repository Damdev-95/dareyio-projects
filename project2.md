## WEB STACK IMPLEMENTATION (LEMP STACK)

This project is similar to the previous stack, but with an alternative Web Server – *NGINX*, which is also very popular and widely used by many websites in the Internet.

## Step 1 – Installing the Nginx Web Server

* Nginx serves as a high performnace web server which is installed using this commands;
```
sudo apt update
sudo apt install nginx
```
* To ensure the succesful installation of the nginx web sever `sudo systemctl status nginx`
* Regarding AWS, the Public IP of the of the nginx web server can be retrieved from the instance using 
``` 
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```
* This shows the default page of the nginx web server

![lamp](https://user-images.githubusercontent.com/71001536/161573075-88fa7c4d-863e-499e-949f-769f803e2b1b.PNG)

## Step 2 — Installing MySQL

* MySQL is a popular relational database management system used within PHP environments, which is used in this project
`sudo apt install mysql-server`
* To enable security measures on the mysql requires an inital secuirty configuration `sudo mysql_secure_installation`
* This shows the mysql displage page on the server

![lamp](https://user-images.githubusercontent.com/71001536/161575336-94d35ea4-df93-4628-a41b-84957efe57f7.PNG)

## Step 3 – Installing PHP
* In order to instal the php to serve dynamic web contents on the nginx server `sudo apt install php-fpm php-mysql`

## Step 4 — Configuring Nginx to Use PHP Processor
![lamp](https://user-images.githubusercontent.com/71001536/161578368-86f9527a-80b6-4a74-aecd-6d8195e0c242.PNG)

![lamp](https://user-images.githubusercontent.com/71001536/162160923-d19dda56-88a6-4688-8ad6-7b3d89ed9a66.PNG)


## Step 5 – Testing PHP with Nginx

![lamp](https://user-images.githubusercontent.com/71001536/162160423-5d7d9f67-a10c-42d7-9f5d-09e0a6abe6d0.PNG)

