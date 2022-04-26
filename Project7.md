## DEVOPS TOOLING WEBSITE SOLUTION

## STEP 1 – PREPARE NFS SERVER

* Spin up a new EC2 instance with RHEL Linux 8 Operating System
* Configure LVM on the server





![image](https://user-images.githubusercontent.com/71001536/164683595-8d899a01-625f-4b48-a01a-c433ff23c7e6.png)


![image](https://user-images.githubusercontent.com/71001536/164688284-058a66ce-e97a-442a-9d89-5cea637a8d9f.png)

```
sudo mkfs.xfs /dev/vg-nfs/lv-apps
sudo mkfs.xfs /dev/vg-nfs/lv-logs
sudo mkfs.xfs /dev/vg-nfs/lv-opt
```
![image](https://user-images.githubusercontent.com/71001536/164692522-0d851f02-e429-4d79-87ee-db0737107186.png)

![image](https://user-images.githubusercontent.com/71001536/164696599-08b07473-b5e3-4fe7-8538-78d5740f97b8.png)

![image](https://user-images.githubusercontent.com/71001536/164697655-c49bd65d-7489-4ec2-b681-8faf3cef55cd.png)

* Install NFS server, configure it to start on reboot and make sure it is u and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

![image](https://user-images.githubusercontent.com/71001536/164699783-aadf62bd-db72-4d8d-bc3c-352759b8fdf4.png)

* Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```
* Configure access to NFS for clients within the same subnet

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
* Then export the filesystem to the clients

`sudo exportfs -arv`

![image](https://user-images.githubusercontent.com/71001536/164701453-fb7140bf-9d81-4986-9cc4-fd0a0837b62c.png)

* Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

![image](https://user-images.githubusercontent.com/71001536/164702126-536e456a-cafd-4381-89c8-deee38e9312c.png)

![image](https://user-images.githubusercontent.com/71001536/164702682-23c60939-eafc-48ff-8055-033926683fb3.png)

## STEP 2 — CONFIGURE THE DATABASE SERVER

`create database tooling`

`CREATE USER 'webaccess'@'%' IDENTIFIED BY 'damilare';`

![image](https://user-images.githubusercontent.com/71001536/164713809-60df0cf3-b3a3-4337-9df8-00ba776c1d0f.png)

```
 grant all privileges on tooling.* to 'webaccess'@'%';
 flush privileges;
 ```
 
##  Step 3 — Prepare the Web Servers
```
#!/bin/bash

sudo yum install nfs-utils nfs4-acl-tools -y
sudo mkdir -p /var/www
sudo mount -t nfs -o rw,nosuid 172.31.19.201:/mnt/apps /var/www
sudo chmod 766 /etc/fstab
sudo echo "172.31.19.201:/mnt/apps /var/www     xfs defaults 0 0" >> /etc/fstab

sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

sudo dnf module reset php -y

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

```
* for automating the installation of the web servers using bash scrpiting , use the bash commands above;

`sudo nano test.sh && sudo chmod +x test.sh`

![image](https://user-images.githubusercontent.com/71001536/164746016-045f32dd-8c35-48cb-95a6-fc4ed3605812.png)

![image](https://user-images.githubusercontent.com/71001536/164746168-453dd093-5db6-4d5e-9318-2a60a9cf4235.png)

![image](https://user-images.githubusercontent.com/71001536/165098739-6f8cde11-13cf-4813-8b75-968006a3caf4.png)

* Move contents of the html folder to the present working directoty
`sudo mv ./html/* .`

* Connect the database to the webservers.
* Edit the bind address in the mysqld.conf

` sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

`sudo systemctl restart mysql`

![image](https://user-images.githubusercontent.com/71001536/165102307-56295862-05f1-4c01-b0d1-66e2f3876376.png)

* Install MySQL client on the WebServer and connect to the MySQL server(DB).
* Ensure to run the connection config in the */var/www/html* directory

```
sudo yum install mysql -y
mysql -u webaccess  -h  172.31.10.110  -pdamilare tooling < tooling-db.sql
mysql -u webaccess  -h  172.31.10.110  -pdamilare tooling 
```


![image](https://user-images.githubusercontent.com/71001536/165238671-97fc3414-a7ca-49a1-be02-d7c5fca907ea.png)

![image](https://user-images.githubusercontent.com/71001536/165238867-27f87999-2759-4daf-b123-e896f9af811b.png)




* Edit the *functions.php* file in the html directory on the web server

`sudo nano functions.php`

![image](https://user-images.githubusercontent.com/71001536/165106463-ec6aceb0-3bd5-4e9a-94f3-07a378882643.png)

* I had no permission  Error – check permissions to your /var/www/html folder and also disable SELinux 
`sudo setenforce 0`

* Edit the file `sudo vim /etc/etc/sysconfig/selinux` and  set SELINUX=disabled

![image](https://user-images.githubusercontent.com/71001536/165115008-0eb4f007-4e3e-4a93-b23b-8d1ff3ca2532.png)

* Initial login with the following credentials
```
username: admin
password: admin
```

![image](https://user-images.githubusercontent.com/71001536/165243356-06003f3c-7635-4313-805d-fdab5982f65e.png)

![image](https://user-images.githubusercontent.com/71001536/165243732-6bb5f599-9029-47ea-aa50-9c9c9c4109d5.png)



