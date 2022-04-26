## LOAD BALANCER SOLUTION WITH APACHE

After completing Project-7 you might wonder how a user will be accessing each of the webservers using 3 different IP addreses or 3 different DNS names

When you have just one Web server and load increases – you want to serve more and more customers, you can add more CPU and RAM or completely replace the server with a more powerful one – this is called "vertical scaling"

Another approach used to cater for increased traffic is "horizontal scaling" – distributing load across multiple Web servers. 

![image](https://user-images.githubusercontent.com/71001536/165248521-7e49a2e8-d5b4-4a13-84f9-889293cd197d.png)

## This is implemented using the Project 7 

* Two RHEL8 Web Servers
* One MySQL DB Server (based on Ubuntu 20.04)
* One RHEL8 NFS server

![image](https://user-images.githubusercontent.com/71001536/165253195-ff07fa98-23e5-402c-9327-ce0dc6fd623e.png)

![image](https://user-images.githubusercontent.com/71001536/165274928-6310ad6b-b65e-4303-a4cf-6bf307cd84db.png)

![image](https://user-images.githubusercontent.com/71001536/165275052-d7695e1b-d0cc-4b92-942f-633a1c18658f.png)

* Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```

* Configure load balancing

`sudo vi /etc/apache2/sites-available/000-default.conf`

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
```
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

![image](https://user-images.githubusercontent.com/71001536/165278967-5560bba0-6b40-4184-9919-ba34cf89d5b0.png)

* The red highlight shows the public ip before load balancer
* The blue highlight shows the private ip of the load balncer

![image](https://user-images.githubusercontent.com/71001536/165279435-8adf2f16-3bff-4043-8b93-e443fb264456.png)

![image](https://user-images.githubusercontent.com/71001536/165280168-44e5df7a-da13-43d4-86be-cf5928ce82e2.png)


*  Configure local domain name resolution. The easiest way is to use */etc/hosts* file, to configure IP address to domain name mapping for our LB
```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

![image](https://user-images.githubusercontent.com/71001536/165281122-c67ec94d-e587-4bbc-bc9c-8345b0248106.png)

![image](https://user-images.githubusercontent.com/71001536/165281413-33c2ff6b-4e91-474e-b2c3-a6686665aa70.png)

## Sites reachable after changing the hosts file

![image](https://user-images.githubusercontent.com/71001536/165281745-5353caf5-31df-46f0-9180-31a6216aa030.png)




