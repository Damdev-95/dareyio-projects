## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

* Create and configure two Linux-based virtual servers (EC2 instances in AWS).
```
Server A name - mysql server
Server B name - mysql client
```

* On mysql server Linux Server install MySQL Server software

`sudo apt install mysql_server`

`sudo mysql_secure_installation`

* On mysql client Linux Server install MySQL Client software

`sudo apt install mysql_client`

* MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. 

* There is a need to change the bind-address from 127.0.0.1 to 0.0.0.0 , to enable the server to allow connections from remote host

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

`sudo systemstl restart mysql`

* Create a user on the mysql server with password for authentication 

`  CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'damilare' ;`

* To extract the  local IP on the mysql server for the security inbound rules

`ip addr show`

* Create a database and grant the remote_user the necessary priviledge

```
CREATE DATABASE example_db;
GRANT ALL ON  example_db.* TO `remote_user`@`%` WITH GRANT OPTION;

```
* From the client, connecting to the server using the remote_user credentials

`sudo mysql -u remote_user -h <private-ip> -p`

![image](https://user-images.githubusercontent.com/71001536/164017691-dfe1cf3e-7115-4e9f-8d18-925a6a622627.png)


