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

 
  
