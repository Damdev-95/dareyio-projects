## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.

This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible Configuration Management, at the same time you will become confident at writing code using declarative language such as YAML

## Ansible Client as a Jump Server (Bastion Host)
A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. If you think about the current architecture you are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

# INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

* Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
* In your GitHub account create a new repository and name it ansible-config-mgt.
* Instal Ansible

```
sudo apt update
sudo apt install ansible
```
![image](https://user-images.githubusercontent.com/71001536/166904274-e101acab-66db-4c94-8b6c-36b8a719b1e7.png)


* Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
* Create a new Freestyle project *ansible* in Jenkins and point it to your ‘ansible-config-mgt’ repository.
* Configure Webhook in GitHub and set webhook to trigger ansible build.
* Configure a Post-build job to save all (**) files, like you did it in Project 9.
* Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

![image](https://user-images.githubusercontent.com/71001536/167273316-20a3ec40-42a6-48d8-811e-160c2d177026.png)


```
sudo apt update
sudo apt install default-jdk-headless

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
* To connect with ssh from the ansible server to the traget hosts using SSH
```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

* Copy the pem files for the web,db,nfs and lb servers to the jump server.
* Assign necessary ownwership to the pem files
```
sudo chown nobody NFS.pem
sudo chown nobody New.pem
```
* ssh into your Jenkins-Ansible server using ssh-agent

```
ssh -A ubuntu@private-ip
```
* Update your inventory/dev.yml file with this snippet of code

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

* Update your playbooks/common.yml file with following code
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
* To execute ansible-playbook command and verify if your playbook actually works:
```
cd ansible-config-mgt
ansible-playbook -i inventory/dev.yaml playbooks/common.yaml
```

![image](https://user-images.githubusercontent.com/71001536/167275076-e75c90ca-f98f-4903-8e6a-004e82d1d553.png)

![image](https://user-images.githubusercontent.com/71001536/167275127-c019eea7-8bce-472d-94cb-2dd0d1c59fe4.png)





