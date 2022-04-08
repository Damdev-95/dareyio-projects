## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

MEAN Stack is a combination of following components:
* MongoDB (Document database) – Stores and allows to retrieve data.
* Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
* Angular (Front-end application framework) – Handles Client and Server Requests
* Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

## Step 1: Install NodeJs
* Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used to set up the Express routes and AngularJS controllers

`sudo apt update`

`sudo apt upgrade`

* It is advisable to download the latest and stable version of the nodejs to avoid compatibility errors
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```
`sudo apt install -y nodejs`

![image](https://user-images.githubusercontent.com/71001536/162425672-e002c329-42a5-48e0-9781-d7f10c2d9129.png)

## Step 2: Install MongoDB

* MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
* Installing the mongoDB using  `sudo apt install -y mongodb`
* Starting the server ` sudo service mongodb start ` and checking status `sudo systemctl status mongodb`

![image](https://user-images.githubusercontent.com/71001536/162421952-f29061e4-88d3-455b-b60e-a6151351b544.png)

## Step 3: Install Express and set up routes to the server

![image](https://user-images.githubusercontent.com/71001536/162420591-79740bea-1cb2-4a1d-b5b2-d1c3534ef778.png)

![image](https://user-images.githubusercontent.com/71001536/162421556-d0391477-6c25-40e9-b997-052c1fedc937.png)

## Step 4 – Access the routes with AngularJS
![image](https://user-images.githubusercontent.com/71001536/162424554-52dc2674-9e9c-41d3-9854-fd78270999a4.png)

![image](https://user-images.githubusercontent.com/71001536/162424201-677c3d09-3aa7-4fce-bde1-69ffca28b5fc.png)



