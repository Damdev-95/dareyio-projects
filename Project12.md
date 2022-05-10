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

