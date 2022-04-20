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



 
  
