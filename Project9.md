## TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.
Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.
n our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/Damdev-95/tooling will be automatically be updated to the Tooling Website.

![image](https://user-images.githubusercontent.com/71001536/165283823-d034968c-6a71-4ed0-83d7-6075a9a63ba6.png)

## Step 1 – Install Jenkins server

* Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
* Install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```
* Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
![image](https://user-images.githubusercontent.com/71001536/165286086-a6e59ebd-8088-40b5-8891-6666c4ad3d06.png)

* By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
![image](https://user-images.githubusercontent.com/71001536/165286395-2de91904-ff58-4601-a98f-74a4fd2f9ea0.png)

![image](https://user-images.githubusercontent.com/71001536/165286651-97356b81-98b3-4e7d-8a7e-a898e28c4aca.png)

* Retrieve it from your server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![image](https://user-images.githubusercontent.com/71001536/165287060-53cd91f3-8fa6-457a-aec1-806be1db118a.png)

**Then you will be asked which plugings to install – choose suggested plugins.**



## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks

* Enable webhooks in your GitHub repository settings

![image](https://user-images.githubusercontent.com/71001536/165288789-886d6808-929e-4aa2-a9bf-574b37526203.png)

* Go to Jenkins web console, click "New Item" and create a "Freestyle project"
![image](https://user-images.githubusercontent.com/71001536/165289479-b1a1924f-c110-4ffe-9957-58fbbd54b9c5.png)

![image](https://user-images.githubusercontent.com/71001536/165289888-31693c58-42fa-49e1-8686-ae875cbbc6a3.png)

![image](https://user-images.githubusercontent.com/71001536/165291196-c8e1173e-0a44-47f3-b483-4272b4bf308f.png)

![image](https://user-images.githubusercontent.com/71001536/165291373-002948fb-a5b8-4a71-ad3f-56c36cf88347.png)

![image](https://user-images.githubusercontent.com/71001536/165291500-5c475f39-af5f-4924-88f7-b423fa5a1ccc.png)

* I had issues with this 

![image](https://user-images.githubusercontent.com/71001536/165514236-24e3ad96-74fa-4283-8770-46cd7c49c86c.png)

* The issues was regarding the ssh-keys on the NFS server

![image](https://user-images.githubusercontent.com/71001536/165910829-62b752ea-3925-4f7a-a8bd-b8285579da32.png)

![image](https://user-images.githubusercontent.com/71001536/165911944-51e17a6b-7fb7-4e4f-b2a5-770fd1e0c885.png)

* Having issues sending the files, there is permission issue on the /mnt/apps folder

![image](https://user-images.githubusercontent.com/71001536/165916745-0c3f4446-1f41-4e16-89d3-7c6ec04dc096.png)

* Change the permission  of the */mnt/apps* directory

`sudo chmod 777 -R /mnt/apps`

![image](https://user-images.githubusercontent.com/71001536/165917311-3fdfed28-73f2-4b08-83ad-ae362e3c6277.png)

* This confirm the changes has been successfully tranfer to the NFS server

![image](https://user-images.githubusercontent.com/71001536/165919992-1a3a7c5c-0a9c-481b-a722-4ed37f72edb5.png)

![image](https://user-images.githubusercontent.com/71001536/165920238-b8668cbe-5482-4c25-8a75-fc895a72c5cc.png)












