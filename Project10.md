## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

## CONFIGURE NGINX AS A LOAD BALANCER
* Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
* Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
* Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
```
sudo apt update
sudo apt install nginx
```
* Open the default nginx configuration file

* Create a load balancer configuration file 

`sudo nano /etc/nginx/sites-available//load_balancer.conf`

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
![image](https://user-images.githubusercontent.com/71001536/165929460-d005842c-1c98-4ca6-8483-ff621b531ba5.png)


* To check the state of nginx , if succesfully configured
 
 `sudo nginx -t`
 
* I made a mistake, I edit the */etc/hosts* in a wrong way
* This really take my time to figure out

![image](https://user-images.githubusercontent.com/71001536/165980865-0676b91b-e4ae-4360-989d-3094d402d0bf.png)


![image](https://user-images.githubusercontent.com/71001536/165980809-c6b96ba5-6850-450a-a712-884fa5f380a2.png)

## My Load balancer is working !!!!!!!!!!!!

![image](https://user-images.githubusercontent.com/71001536/165981060-70af1d35-57a3-46dd-9e6a-8bc60a5fc5d3.png)

![image](https://user-images.githubusercontent.com/71001536/165981267-1ee2f60e-340a-4681-a1bf-d88692378091.png)

![image](https://user-images.githubusercontent.com/71001536/165981343-d11cd415-c66c-4478-8320-c9adb181b6f5.png)

![image](https://user-images.githubusercontent.com/71001536/165981505-37b77401-68bf-46da-8f56-57612e380e52.png)




