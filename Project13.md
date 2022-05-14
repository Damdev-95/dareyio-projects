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

