## WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS
A technology stack is a set of frameworks and tools used to develop a software product. This set of frameworks and tools are very specifically chosen to work together in creating a well-functioning software.

**LAMP**  (Linux, Apache, MySQL, PHP or Python, or Perl)

## STEP 1
* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

![lamp](https://user-images.githubusercontent.com/71001536/161437459-58039884-6bc9-4c44-9b84-dfb8f958ba8c.PNG)

* Use the public IP on the instance for remote login on the instance
* For my server login, I used Mobaxterm for remote administartion
![lamp](https://user-images.githubusercontent.com/71001536/161437850-2756b3c9-14bd-46df-8123-9b2942c7ece3.PNG)

* install Apache2 web server,MySql server for database and PhP for the backend development

![lamp](https://user-images.githubusercontent.com/71001536/161438834-cadfd39d-9e54-4dd0-ae0e-cf6b75f78570.PNG)
* Changed the default directory to index.php

![lamp](https://user-images.githubusercontent.com/71001536/161439112-8873cc65-a8c3-41ad-93e8-a9225be04176.PNG)
