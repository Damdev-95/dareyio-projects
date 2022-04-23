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

![image](https://user-images.githubusercontent.com/71001536/164731933-67b30843-ffaa-4392-a144-b693b93fa587.png)


![image](https://user-images.githubusercontent.com/71001536/164746016-045f32dd-8c35-48cb-95a6-fc4ed3605812.png)

![image](https://user-images.githubusercontent.com/71001536/164746168-453dd093-5db6-4d5e-9318-2a60a9cf4235.png)

