## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

In this project you will continue working with ansible-config-mgt repository and make some improvements of your code. Now you need to refactor your Ansible code, create assignments, and learn how to use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows you to organize your tasks and reuse them when needed.

# Code Refactoring
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity, add proper comments without affecting the logic.

# Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job we will require Copy Artifact plugin.

* Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
```
sudo mkdir /home/ubuntu/ansible-config-artifact
sudo chmod -R 0777 /home/ubuntu/ansible-config-artifact
```

* Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![image](https://user-images.githubusercontent.com/71001536/167598156-b9e4baa7-e000-4245-9e86-8c72b7d94084.png)

* Create a new Freestyle project (you have done it in Project 9) and name it *save_artifacts*

* This project will be triggered by completion of your existing ansible project. Configure it accordingly:

![image](https://user-images.githubusercontent.com/71001536/167599782-ab17bd13-17bd-4f88-8bc8-1300136667f4.png)

![image](https://user-images.githubusercontent.com/71001536/167599964-408d0afe-671e-4a2d-9221-d323ea2cb650.png)

* The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.

![image](https://user-images.githubusercontent.com/71001536/167600784-4c0b4726-5cf0-4379-9777-a21cdc4bd73d.png)

* I encounter a build error, regarding permission denied 
![image](https://user-images.githubusercontent.com/71001536/167605195-dd6a84a1-8289-4445-ba39-49139750adbc.png)

* I resolved it by giving the necessary permission

`sudo chmod 0755 /home/ubuntu`

![image](https://user-images.githubusercontent.com/71001536/167605736-1a5f9e1a-07a5-476e-b8c3-a475db159c09.png)

![image](https://user-images.githubusercontent.com/71001536/167605954-e25f0f15-c8ae-4cf5-baf8-aa6416339ccd.png)

# Step 2 – Refactor Ansible code by importing other playbooks into site.yaml

Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.
Let see code re-use in action by importing other playbooks

* Within playbooks folder, create a new file and name it site.yaml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. 

* Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.

* Move common.yaml file into the newly created static-assignments folder.

![image](https://user-images.githubusercontent.com/71001536/167611202-015a2c94-e886-48b3-ac24-5e3fa8e554e5.png)


* Inside site.yaml file, import common.yml playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yaml
```
* Run ansible-playbook command against the dev environment

`ansible-playbook -i inventory/dev.yaml playbooks/site.yaml`

## Had issue trying to use the site.yaml file 

![image](https://user-images.githubusercontent.com/71001536/167614229-8c4d4183-509e-4c82-b48b-8905c508eb9d.png)

## Solve renaming the yml with yaml , with necessary chmod permission


* Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yaml. In this playbook, configure deletion of wireshark utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

![image](https://user-images.githubusercontent.com/71001536/167617214-f58eb0bb-a7e1-48e9-a5fb-ae6bc93a9b0b.png)

* Update site.yaml with - import_playbook: ../static-assignments/common-del.yaml instead of common.yaml and run it against dev servers:

![image](https://user-images.githubusercontent.com/71001536/167617682-44c7ef42-9353-4dab-a70e-2e1690ab7fee.png)

![image](https://user-images.githubusercontent.com/71001536/167617964-c3c7f0a4-58ee-4ff8-ba30-85b8207c8c54.png)

* To validate the removal of wiresharks from the target hosts

![image](https://user-images.githubusercontent.com/71001536/167618481-f06f6fa9-7f37-4607-bca4-bc9b22343b0a.png)

![image](https://user-images.githubusercontent.com/71001536/167618560-6e05cd2a-a4c3-4049-87a7-841125b29731.png)

# Step 3 – Configure UAT Webservers with a role ‘Webserver’

* Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

* To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.

```
sudo mkdir roles
cd roles
sudo ansible-galaxy init webserver
```
![image](https://user-images.githubusercontent.com/71001536/167620984-8101df76-01ba-4c6b-86fa-43eac1524cb9.png)

![image](https://user-images.githubusercontent.com/71001536/168417315-11b5315f-1721-44a2-8814-9bdf3969250d.png)


* Update your inventory ansible-config-mgt/inventory/uat.yaml file with IP addresses of your 2 UAT Web servers
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```
![image](https://user-images.githubusercontent.com/71001536/167621993-28a28114-0094-47fd-b682-2e391442e309.png)

* In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles
## ERROR, I did not find the /etc/ansible folder 

* I have to uninstall the current ansible, then install latest

```
sudo apt remove ansible
sudo apt --purge autoremove
sudo apt update
sudo apt upgrade
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
ansible --version
sudo apt install python3-argcomplete
sudo activate-global-python-argcomplete3
```

![image](https://user-images.githubusercontent.com/71001536/167633222-18e91773-c29d-4181-b50c-deb5b8255bfa.png)


* In */etc/ansible/ansible.cfg* file uncomment roles_path string and provide a full path to your roles directory roles_path    = */home/ubuntu/ansible-config-mgt/roles*, so Ansible could know where to find configured roles.

* It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started

  
  
* Your main.yml may consist of following tasks:
  
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/damdev-95/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```
# Step 4 – Reference ‘Webserver’ role
  
* Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

* There is need to refer the *uat-webservers.yaml* role inside *site.yaml*.

  
```
  ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yaml

  ```
  
  # Step 5 – Commit & Test

* Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

Now run the playbook against your uat inventory :

`ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yaml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml`
 
![image](https://user-images.githubusercontent.com/71001536/168425301-20a35738-a543-46cc-98d9-77b0714b10bf.png)

# UAT WEBSERVER 1
![image](https://user-images.githubusercontent.com/71001536/168425468-1d2a96fc-5e19-4a6a-a5be-119f88a92151.png)


 # UAT WEBSERVER 2

![image](https://user-images.githubusercontent.com/71001536/168425525-3f40962c-7279-4b70-8170-745212bdb93a.png)


