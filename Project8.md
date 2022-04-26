## LOAD BALANCER SOLUTION WITH APACHE

After completing Project-7 you might wonder how a user will be accessing each of the webservers using 3 different IP addreses or 3 different DNS names

When you have just one Web server and load increases – you want to serve more and more customers, you can add more CPU and RAM or completely replace the server with a more powerful one – this is called "vertical scaling"

Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. 

![image](https://user-images.githubusercontent.com/71001536/165248521-7e49a2e8-d5b4-4a13-84f9-889293cd197d.png)

## This is implemented using the Project 7 
Two RHEL8 Web Servers
One MySQL DB Server (based on Ubuntu 20.04)
One RHEL8 NFS server

![image](https://user-images.githubusercontent.com/71001536/165253195-ff07fa98-23e5-402c-9327-ce0dc6fd623e.png)
