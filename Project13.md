##  ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

Last 2 projects have already equipped you with some knowledge and skills on Ansible, so you can perform configurations using playbooks, roles and imports. Now you will continue configuring your UAT servers learning and practicing new Ansible concepts and modules.

In this project we will introduce dynamic assignments by using include module.

Now you may be wondering, what is the difference between static and dynamic assignments?

Well, from Project 12, you can already tell that static assignments use import Ansible module. The module that enables dynamic assignments is include.

```
import = Static
include = Dynamic
```
# Introducing Dynamic Assignment Into Our structure

* Create a new branch in the *ansible-config-mgt* github workflow with the name *dynamic-assignments*
* Create a directory *dynamic-assignments* in the *ansible-config-mgt* folder Then inside this folder, create a new file and name it *env-vars.yml*. We will instruct *site.yml* to include this playbook later. For now, let us keep building up the structure.

```
mkdir dynamic-assignments
sudo nano env-vars.yaml
```
* Ensure the folder and yaml file  has ubuntu ownwership with 775 chmod permission

![image](https://user-images.githubusercontent.com/71001536/168434093-c26eac0c-d89b-48c6-8510-69dbe7c2d86a.png)

* Copy the text below into the *env-vars.yaml*


```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yaml
            - stage.yaml
            - prod.yaml
            - uat.yaml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always


```

Notice 3 things to notice here:

* We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
include_role
include_tasks
include_vars
In the same version, variants of import were also introduces, such as:

import_role
import_tasks

* We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

* We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
  
  ```
  * Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql
 
 ```
 cd /home/ubuntu/ansible-config-mgt/roles
 sudo ansible-galaxy install geerlingguy.mysql
 sudo mv geerlingguy.mysql/ mysql
 ```

* Upload the changes into my GitHub

```
git add .
git commit -m "Commit new role files into GitHub"
# git push --set-upstream origin roles-feature  (old features to push)
git push https://<github-token>/Damdev-95/ansible-config-mgt.git
```
* Install the apache and nginx role from the ansible galaxy 

```
cd /home/ubuntu/ansible-config-mgt/roles
sudo ansible-galaxy install geerlingguy.nginx
sudo mv geerlingguy.nginx/ nginx
sudo ansible-galaxy install geerlingguy.apache
sudo mv geerlingguy.apache/ apache
sudo chown ubuntu apache nginx
sudo chmod 775 apache nginx
```

# Important Hints:

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively


![image](https://user-images.githubusercontent.com/71001536/168632740-b14aa1f4-cdb2-416c-8353-0305e908c011.png)

![image](https://user-images.githubusercontent.com/71001536/168632902-469445ed-5b89-44de-b6bb-caf27ed0bb35.png)

