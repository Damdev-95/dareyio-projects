## IMPLEMENT A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS)

* Create and configure two Linux-based virtual servers (EC2 instances in AWS).

* On mysql server Linux Server install MySQL Server software
`sudo apt install mysql_server

* On mysql client Linux Server install MySQL Client software
`sudo apt install mysql_client`

* MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. 
* There is a need to configure to server to allow connections from remote host
`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

* Create a user on the mysql with password for authentication
`  CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'damilare' `

