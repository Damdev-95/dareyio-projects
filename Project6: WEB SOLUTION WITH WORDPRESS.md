## WEB SOLUTION WITH WORDPRESS

In this project, it is tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress.
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

## Three-tier Architecture
Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.
* Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
* Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
* Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. 
  Database Server or File System Server such as FTP server, or NFS Server
  
  ## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.
 
![image](https://user-images.githubusercontent.com/71001536/164032944-74deb8a0-ba22-4d66-8733-69f96dc2c82e.png)
 
![image](https://user-images.githubusercontent.com/71001536/164033917-f9d4e8b9-68d9-4060-aa55-5e2c24356ded.png)

* Login to the web server and use `lsblk` command to inspect what block devices are attached to the server
![image](https://user-images.githubusercontent.com/71001536/164224573-9b2673f8-4b47-4fd8-8fc2-92d7904f561a.png)

* Creating partition on the disk and  gdisk utility to create a single partition on each of the 3 disks
```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```
![image](https://user-images.githubusercontent.com/71001536/164227160-4c236de5-8fd7-44cb-923e-d6817b414492.png)

![image](https://user-images.githubusercontent.com/71001536/164227265-43827f7b-bb68-45e6-9531-7a7b456533db.png)

![image](https://user-images.githubusercontent.com/71001536/164228016-11eae77c-6721-48b5-a7f8-1b1c62fcee76.png)
* Install lvm2 on the web server using `sudo yum instal lvm2 -y`
* 
* physical volume is can only be create on the 3 partition disks, which is done using the `pvcreate` utlity to be used by lvm

```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1
```
* Verify that the Physical volume has been created successfully by running `sudo pvs`

![image](https://user-images.githubusercontent.com/71001536/164229876-89fb3e7c-ce25-402d-8b1c-a48643332045.png)
* Use `vgcreate` utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg and verify the sucessfully created volume group using `sudo vgs`
```
sudo vgcreate vg-webdata /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
![image](https://user-images.githubusercontent.com/71001536/164231353-6676f0da-49fe-4b3c-b11b-7f9f275cfad2.png)

* Use `lvcreate` utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G vg-webdata
sudo lvcreate -n logs-lv -L 14G vg-webdata
```


![image](https://user-images.githubusercontent.com/71001536/164232617-f700addc-5dc1-4f3c-9082-928c244e2ca4.png)

![image](https://user-images.githubusercontent.com/71001536/164245803-0f048756-f35e-4930-87c5-bdebad255a61.png)
* Attaching another */dev/xvdi*, create a physical volume, add to vg-webdata , then
`sudo lvx -n apps-lv -L 25G vg-webdata`

* Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
 sudo mkfs.ext4 /dev/vg-webdata/apps-lv
 sudo mkfs.ext4 /dev/vg-webdata/logs-lv

```

* Create /var/www/html directory to store website files `sudo mkdir -p /var/www/html`

* Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`

* Mount /var/www/html on apps-lv logical volume 
`sudo mount /dev/vg-webdata/apps-lv /var/www/html/`

* Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`

* Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is the previous step is very
important)
`sudo mount /dev/vg-webdata/logs-lv /var/log`

* Restore log files back into /var/log directory 
`sudo rsync -av /home/recovery/logs/. /var/log`

![image](https://user-images.githubusercontent.com/71001536/164255043-5f07dbc1-4da2-4def-b9c5-e0866c319698.png)
* check the UUID of the volume 
![image](https://user-images.githubusercontent.com/71001536/164262022-0cc4095a-47b7-4dc9-bfbc-e0d33b29136e.png)


* Update /etc/fstab file so that the mount configuration will persist after restart of the server
```
sudo vim /etc/fstab
```

![image](https://user-images.githubusercontent.com/71001536/164260840-1a32c960-dfc1-414e-9ee9-4161d3b510a2.png)

* Test the configuration and reload the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
![image](https://user-images.githubusercontent.com/71001536/164261579-2d158e3c-6112-43a0-bb24-c08984e69e0e.png)

## Step 2 — Prepare the Database Server

* Use the RedHat Linux for the database server.
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

![image](https://user-images.githubusercontent.com/71001536/164423344-85c16ae4-cba5-40c7-bd0f-5a15f24663d9.png)

`sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

`sudo lvcreate -n db-lv -L 20G vg-database`

`sudo mkfs.ext4 /dev/vg-database/db-lv`

![image](https://user-images.githubusercontent.com/71001536/164425302-42d6d0a4-add4-44be-ae18-05757895609f.png)

![image](https://user-images.githubusercontent.com/71001536/164430304-a7fcf43f-2d0e-415d-b9f4-ac5b7a02c2f5.png)




## Step 3 — Install WordPress on your Web Server EC2

* Update the repository 
`sudo yum -y update`
* Install wget, Apache and it’s dependencies
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

* Start Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
* To install PHP and it’s depemdencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
* Restart Apache `sudo systemctl restart httpd`

* Download wordpress and copy wordpress to *var/www/html*
```
mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  ```
* Configure SELinux Policies
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

## Step 4 — Install MySQL on your DB Server EC2
* This step include the installation of mysql-server on the Database server
```
sudo yum update
sudo yum install mysql-server
```
* Verify that the service is up and running by using
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

![image](https://user-images.githubusercontent.com/71001536/164432458-32d12089-3103-4b9c-ac79-6c3e4da1c176.png)


## Step 5 — Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`172.31.5.140` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.5.140';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

Using `ip addr show` to check the private ip on the web server.

![image](https://user-images.githubusercontent.com/71001536/164433422-89a5f91e-4112-40f6-b970-d26d5ab294ce.png)

## Step 6 — Configure WordPress to connect to remote database.

![image](https://user-images.githubusercontent.com/71001536/164440215-e4e64674-c19c-4745-abbd-0f85b1c49335.png)

## ENCOUNTER THIS ERROR
![image](https://user-images.githubusercontent.com/71001536/164441501-d01c7a5d-5ed9-4085-83d9-5f9c96f2604a.png)

* This requires to debug and troubleshoot
* The issue is from the WordPress database credentials are stored in the wp-config.php file
* adding the bind-address (0.0.0.0) in the */etc/my.cnf*
`sudo vim /etc/my.cnf`
`sudo systemctl restart mysqld`

![image](https://user-images.githubusercontent.com/71001536/164461928-d65818c2-a63e-4b68-8ce4-c9f113ecc188.png)

![image](https://user-images.githubusercontent.com/71001536/164462630-61a09584-ee1a-4bd3-a92b-bc43ad585809.png)

![image](https://user-images.githubusercontent.com/71001536/164463389-6a49589e-0d52-4618-9edd-c9099c6fe0f9.png)

![image](https://user-images.githubusercontent.com/71001536/164463529-510e08b3-4445-43bf-8c37-54416d0d0ff2.png)



