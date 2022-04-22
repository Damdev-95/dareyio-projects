## WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS
A technology stack is a set of frameworks and tools used to develop a software product. This set of frameworks and tools are very specifically chosen to work together in creating a well-functioning software.

**LAMP**  (Linux, Apache, MySQL, PHP or Python, or Perl)

## Step 1 — installing apache and updating the firewall

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

![lamp](https://user-images.githubusercontent.com/71001536/161437459-58039884-6bc9-4c44-9b84-dfb8f958ba8c.PNG)

* Use the public IP on the instance for remote login on the Linux server.
* For my server login, I used Mobaxterm for remote administration.
* Install Apache using Ubuntu’s package manager ‘apt’:
```
#update a list of packages in package manager
sudo apt update
#run apache2 package installation
sudo apt install apache2
```

![lamp](https://user-images.githubusercontent.com/71001536/161437850-2756b3c9-14bd-46df-8123-9b2942c7ece3.PNG)

* To verify that apache2 is running as a Service in our OS, use following command
`sudo systemctl status apache2`

* To retrive the public-ip from the instance.
`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`

## Step 2 — installing mysql and php
* Install MySql server for database and PhP for the backend development
```
sudo apt install mysql-server
sudo mysql_secure_installation
```

* Installing php will involves the apache, mysql packages for the php environment, To install these 3 packages at once
`sudo apt install php libapache2-mod-php php-mysql`

* Once the installation is finished, you can run the following command to confirm your PHP version: `php -v`

## Step 3 — creating a virtual host for your website using apache

* Create the directory for projectlamp using *mkdir* command as follows
`sudo mkdir /var/www/projectlamp`

* Assign ownership of the directory with your current system user
`sudo chown -R $USER:$USER /var/www/projectlamp`

* create and open a new configuration file in Apache’s sites-available directory
`sudo vi /etc/apache2/sites-available/projectlamp.conf`

* Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text
```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* To show the new file in the sites-available directory
`sudo ls /etc/apache2/sites-available`

![image](https://user-images.githubusercontent.com/71001536/164651614-350561cc-d871-4f26-928a-6b5e92f83b29.png)

* Use *a2ensite* command to enable the new virtual host:

`sudo a2ensite projectlamp`

* To disable Apache’s default website use *a2dissite* command

`sudo a2dissite 000-default`

* To make sure your configuration file doesn’t contain syntax errors and reload Apache so these changes take effect
 ```
 sudo apache2ctl configtest
 sudo systemctl reload apache2
 ```
 * Create an index.html file in that location */var/www/projectlamp/* so that we can test that the virtual host works
 ```
 sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
 ```
# The >> redirection operator will append lines to the end of the specified file

# The > redirection operator will empty and overwritethe specified file

* 0pen the website URL using IP address  `http://<Public-IP-Address>:80`

![lamp](https://user-images.githubusercontent.com/71001536/161438834-cadfd39d-9e54-4dd0-ae0e-cf6b75f78570.PNG)

## Step 4 — Enable php on the website

* There is need to edit the */etc/apache2/mods-enabled/dir.conf* file and change the order in which the index.php file is listed within the DirectoryIndex directive:

`sudo vim /etc/apache2/mods-enabled/dir.conf`

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

* Saving and closing the file, there is need to reload Apache so the changes take effect

`sudo systemctl reload apache2`

* Create a new file named *index.php* inside your custom web root folder */var/www/projectlamp/*:

`vim /var/www/projectlamp/index.php`

```
<?php
phpinfo();
```

![lamp](https://user-images.githubusercontent.com/71001536/161439112-8873cc65-a8c3-41ad-93e8-a9225be04176.PNG)

* It’s best to remove the file you created as it contains sensitive information about your PHP environment -and your Ubuntu server.

`sudo rm /var/www/projectlamp/index.php`

Applications of Lampstack  such *WordPress, Drupal*
