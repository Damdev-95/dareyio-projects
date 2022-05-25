## EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP

What is Continuous Integration?
In software engineering, Continuous Integration (CI) is a practice of merging all developers’ working copies to a shared mainline (e.g., Git Repository or some other version control system) several times per day. Frequent merges reduce chances of any conflicts in code and allow to run tests more often to avoid massive rework if something goes wrong. This principle can be formulated as Commit early, push often.

The general idea behind multiple commits is to avoid what is generally considered as Merge Hell or Integration hell. When a new developer joins a new project, he or she must create a copy of the main codebase by starting a new feature branch from the mainline to develop his own features (in some organization or team, this could be called a develop, main or master branch). If there are tens of developers working on the same project, they will all have their own branches created from mainline at different points in time. Once they make a copy of the repository it starts drifting away from the mainline with every new merge of other developers’ codes. If this lingers on for a very long time without reconciling the code, then this will cause a lot of code conflict or Merge Hell, as rightly said. Imagine such a hell from tens of developers or worse, hundreds. So, the best thing to do, is to continuously commit & push your code to the mainline. As many times as tens times per day. With this practice, you can avoid Merge Hell or Integration hell

# CI concept is not only about committing your code. There is a general workflow, let us start it…

* Run tests locally: Before developers commit their code to a central repository, it is recommended to test the code locally. So, Test-Driven Development (TDD) approach is commonly used in combination with CI. Developers write tests for their code called unit-tests, and before they commit their work, they run their tests locally. This practice helps a team to avoid having one developer’s work-in-progress code from breaking other developers’ copy of the codebase.

* Compile code in CI: After testing codes locally, developers commit and push their work to a central repository. Rather than building the code into an executable locally, a dedicated CI server picks up the code and runs the build there. In this project we will use, already familiar to you, Jenkins as our CI server. Build happens either periodically – by polling the repository at some configured schedule, or after every commit. Having a CI server where builds run is a good practice for a team, as everyone has visibility into each commit and its corresponding builds.

* Run further tests in CI: Even though tests have been run locally by developers, it is important to run the unit-tests on the CI server as well. But, rather than focusing solely on unit-tests, there are other kinds of tests and code analysis that can be run using CI server. These are extremely critical to determining the overall quality of code being developed, how it interacts with other developers’ work, and how vulnerable it is to attacks. A CI server can use different tools for Static Code Analysis, Code Coverage Analysis, Code smells Analysis, and Compliance Analysis. In addition, it can run other types of tests such as Integration and Penetration tests. Other tasks performed by a CI server include production of code documentation from the source code and facilitate manual quality assurance (QA) testing processes.

* Deploy an artifact from CI: At this stage, the difference between CI and CD is spelt out. As you now know, CI is Continuous Integration, which is everything we have been discussing so far. CD on the other hand is Continuous Delivery which ensures that software checked into the mainline is always ready to be deployed to users. The deployment here is manually triggered after certain QA tasks are passed successfully. There is another CD known as Continuous Deployment which is also about deploying the software to the users, but rather than manual, it makes the entire process fully automated. Thus, Continuous Deployment is just one step ahead in automation than Continuous Delivery.

* Version Control: This is the stage where developers’ code gets committed and pushed after they have tested their work locally.
Build: Depending on the type of language or technology used, we may need to build the codes into binary executable files (in case of compiled languages) or just package the codes together with all necessary dependencies into a deployable package (in case of interpreted languages).

* Unit Test: Unit tests that have been developed by the developers are tested. Depending on how the CI job is configured, the entire pipeline may fail if part of the tests fails, and developers will have to fix this failure before starting the pipeline again. A Job by the way, is a phase in the pipeline. Unit Test is a phase, therefore it can be considered a job on its own.
* Deploy: Once the tests are passed, the next phase is to deploy the compiled or packaged code into an artifact repository. This is where all the various versions of code including the latest will be stored. The CI tool will have to pick up the code from this location to proceed with the remaining parts of the pipeline.

* Auto Test: Apart from Unit testing, there are many other kinds of tests that are required to analyse the quality of code and determine how vulnerable the software will be to external or internal attacks. These tests must be automated, and there can be multiple environments created to fulfil different test requirements. For example, a server dedicated for Integration Testing will have the code deployed there to conduct integration tests. Once that passes, there can be other sub-layers in the testing phase in which the code will be deployed to, so as to conduct further tests. Such are User Acceptance Testing (UAT), and another can be Penetration Testing. These servers will be named according to what they have been designed to do in those environments. A UAT server is generally be used for UAT, SIT server is for Systems Integration Testing, PEN Server is for Penetration Testing and they can be named whatever the naming style or convention in which the team is used. An environment does not necessarily have to reside on one single server. In most cases it might be a stack as you have defined in your Ansible Inventory. All the servers in the inventory/dev are considered as Dev Environment. The same goes for inventory/stage (Staging Environment) inventory/preprod (Pre-production environment), inventory/prod (Production environment), etc. So, it is all down to naming convention as agreed and used company or team wide.

* Deploy to production: Once all the tests have been conducted and either the release manager or whoever has the authority to authorize the release to the production server is happy, he gives green light to hit the deploy button to ship the release to production environment. This is an Ideal Continuous Delivery Pipeline. If the entire pipeline was automated and no human is required to manually give the Go decision, then this would be considered as Continuous Deployment. Because the cycle will be repeated, and every time there is a code commit and push, it causes the pipeline to trigger, and the loop continues over and over again.

* Measure and Validate: This is where live users are interacting with the application and feedback is being collected for further improvements and bug fixes. There are many metrics that must be determined and observed here. We will quickly go through 13 metrics that MUST be considered. 

# Common Best Practices of CI/CD
Before we move on to observability metrics – let us list down the principles that define a reliable and robust CI/CD pipeline:

Maintain a code repository
Automate build process
Make builds self-tested
Everyone commits to the baseline every day
Every commit to baseline should be built
Every bug-fix commit should come with a test case
Keep the build fast
Test in a clone of production environment
Make it easy to get the latest deliverables
Everyone can see the results of the latest build
Automate deployment (if you are confident enough in your CI/CD pipeline and willing to go for a fully automated Continuous Deployment)

# SIMULATING A TYPICAL CI/CD PIPELINE FOR A PHP BASED APPLICATION
As part of the ongoing infrastructure development with Ansible started from Project 11, you will be tasked to create a pipeline that simulates continuous integration and delivery. Target end to end CI/CD pipeline is represented by the diagram below. It is important to know that both Tooling and TODO Web Applications are based on an interpreted (scripting) language (PHP). It means, it can be deployed directly onto a server and will work without compiling the code to a machine language.

The problem with that approach is, it would be difficult to package and version the software for different releases. And so, in this project, we will be using a different approach for releases, rather than downloading directly from git, we will be using Ansible uri module.
![image](https://user-images.githubusercontent.com/71001536/169779907-e84dafbe-bee4-40c8-bbb2-70baec9243eb.png)


![image](https://user-images.githubusercontent.com/71001536/169774423-f01ca2b3-aa7e-4199-9a94-c4ff2e0dd55b.png)

![image](https://user-images.githubusercontent.com/71001536/169796345-324c336f-ccee-4ff0-8674-4c4f62efa20e.png)

# Installing Ansible and dependencies for RedHat 
```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install java-11-openjdk-devel -y
sudo yum install jenkins
sudo systemctl daemon-reload
```
# Updating the bash profile on the RedHat Server

*  open the bash profile 
`vi .bash_profile `

* paste the below in the bash profile
```
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

* Reload the bash profile
`source ~/.bash_profile`

![image](https://user-images.githubusercontent.com/71001536/169805436-b85a87eb-ebaf-45f1-b99b-5d9a0ab98ac2.png)

![image](https://user-images.githubusercontent.com/71001536/169805632-16d5ca4e-e4d1-4c91-b949-064c2d72d34a.png)

* Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory

![image](https://user-images.githubusercontent.com/71001536/169806230-27924877-5569-4fb1-b7d6-bb405845fdb9.png)

* Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```
*  Now go back into the Ansible pipeline in Jenkins, and select *configure*, Scroll down to Build Configuration section and specify the location of the Jenkinsfile at *deploy/Jenkinsfile*

![image](https://user-images.githubusercontent.com/71001536/169808776-07c79b14-b794-42c5-9644-705e463fbef9.png)

![image](https://user-images.githubusercontent.com/71001536/169813175-94fbcc70-c03b-4d96-a4d8-a419de28760d.png)


* Create a new git branch and name it *feature/jenkinspipeline-stages*

* Currently we only have the Build stage. Let us add another stage called *Test*. Paste the code snippet below and push the new changes to GitHub.

```
pipeline {
    agent any

  stages {

    stage("Initial cleanup") {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Package') {
      steps {
        script {
          sh 'echo "Packaging Stage"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploying Stage"'
        }
      }
    }

    stage('Clean up') {
      steps {
        cleanWs()
      }
    }
  }
}

```
* To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

* Click on the "Administration" button

* Navigate to the Ansible project and click on "Scan repository now"

![image](https://user-images.githubusercontent.com/71001536/170229285-0baf6a07-6dab-4fd0-89ff-4b9717db5d56.png)

![image](https://user-images.githubusercontent.com/71001536/170229809-b99bdf3c-a166-46e4-95ae-d54a50cdfab4.png)

# Both the branch and main have build and done corrrectly
![image](https://user-images.githubusercontent.com/71001536/170230275-4a637a38-ffc5-49b1-99ed-7509444b336c.png)

# Installing Ansible and dependices
```
sudo yu install ansible -y
python3 -m pip install --upgrade setuptools
python3 -m pip install --upgrade pip
python3 -m pip install PyMySQL
python3 -m pip install mysql-connector-python
python3 -m pip install psycopg2==2.7.5 --ignore-installed
```

# Ansible dependencies

* For Mysql Database
`ansible-galaxy collection install community.mysql`
* For Postgresql Database
`ansible-galaxy collection install community.postgresql`

* Install ANSIBLE plugin on Jenkins
* There is need to export some variable
* create *ansible.cfg* in the deploy directory
```
[defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true


[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
```
* Create a new jenkins file for executing ansible

```
pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }

  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/Damdev-95/ansible-config-mgt.git'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yaml'
         }
      }

      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }

}
```
* Create credentials for the ansible 
![image](https://user-images.githubusercontent.com/71001536/170238305-18f5e8d7-579d-4fb2-8eca-7b8083867db5.png)

![image](https://user-images.githubusercontent.com/71001536/170241486-57357faa-f537-4ffd-a982-07ca2122774b.png)

![image](https://user-images.githubusercontent.com/71001536/170241756-242a521f-00f1-4fe7-8bb1-ef57989a2cb9.png)

![image](https://user-images.githubusercontent.com/71001536/170241922-26bd47b8-c2c2-4237-b873-e62a8f25c6b1.png)

![image](https://user-images.githubusercontent.com/71001536/170244084-e86a22e9-3ed4-4aed-bd02-98ebbc0c1306.png)

![image](https://user-images.githubusercontent.com/71001536/170250268-642db162-232b-4422-88da-dd5814e30fb6.png)

![image](https://user-images.githubusercontent.com/71001536/170250393-ce7cbd85-4e8a-4e51-8005-e6cd12d39f8c.png)

![image](https://user-images.githubusercontent.com/71001536/170253389-5eb650a3-6386-4d87-b1c5-5366f6ddbc9a.png)

![image](https://user-images.githubusercontent.com/71001536/170250661-ce84851d-b984-4719-9494-8c5481430933.png)

# CI/CD PIPELINE FOR TODO APPLICATION

We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04

# Phase 1 – Prepare Jenkins

* Fork the repository below into your GitHub account

`https://github.com/darey-devops/php-todo.git`

* On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)
`sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}`

* Install Jenkins plugins
* Plot plugin
* Artifactory plugin

![image](https://user-images.githubusercontent.com/71001536/170276625-ec121853-3e26-4ab2-a5b4-7edaf194d1ce.png)

## open port 8082 on the artifactory server

![image](https://user-images.githubusercontent.com/71001536/170277919-1b78cfe7-4c23-41f8-a7cc-370d61cc864a.png)

![image](https://user-images.githubusercontent.com/71001536/170278543-c4ea5ed1-3b54-4549-a39a-30ec5da99717.png)

* Create local repository
![image](https://user-images.githubusercontent.com/71001536/170278834-d90573c1-2567-4d18-942b-6374e8a2752f.png)

![image](https://user-images.githubusercontent.com/71001536/170280426-ecd2af7a-71a9-4ec6-9d15-40ecef3ec10a.png)

![image](https://user-images.githubusercontent.com/71001536/170280641-cfc58754-0971-40f2-b349-e1ba4e9432b8.png)

## ip address of the jenkins server in the homestead database

![image](https://user-images.githubusercontent.com/71001536/170281713-be871fa1-41e7-4f9e-9361-e827c8c8ea9d.png)

![image](https://user-images.githubusercontent.com/71001536/170283253-d3923359-4434-4acd-8538-e1939acf2e6b.png)

![image](https://user-images.githubusercontent.com/71001536/170283901-710386f4-ec14-437a-a3c3-cf660cceb054.png)

# Edit bind address on the db server 
` sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf `

# Edit the env.sample file in the php-todo

![image](https://user-images.githubusercontent.com/71001536/170291044-d864d5e1-72be-4486-b5a6-ff309fb36178.png)

![image](https://user-images.githubusercontent.com/71001536/170291770-1566c7fe-a7e8-47a2-8595-6ab210d4af7c.png)

![image](https://user-images.githubusercontent.com/71001536/170295948-e98f0709-4ae2-413b-831d-3e560d9b9ee6.png)

## Phase 3 – Code Quality Analysis

![image](https://user-images.githubusercontent.com/71001536/170299391-6445cbcd-b1b5-4476-9688-09f557af6d6c.png)
![image](https://user-images.githubusercontent.com/71001536/170312391-3f6d9dd1-5a35-40f3-ad71-977e1c29f622.png)

![image](https://user-images.githubusercontent.com/71001536/170313258-96dee1da-4d64-4b77-a76e-59ebccf893df.png)

![image](https://user-images.githubusercontent.com/71001536/170316569-61043f74-0d7a-43a1-adeb-9f90df9d8096.png)

![image](https://user-images.githubusercontent.com/71001536/170317063-446fdd8d-02be-4524-8b77-05f1459ad7a5.png)


## SONARQUBE INSTALLATION

* Using the sonarqube role for the ansible installation
![image](https://user-images.githubusercontent.com/71001536/170323219-ef735159-7bae-4bd7-a7bc-0e3e128a4c69.png)

![image](https://user-images.githubusercontent.com/71001536/170323722-976eee7d-b117-4519-be76-32fb3e87da61.png)

![image](https://user-images.githubusercontent.com/71001536/170323977-17ba1de4-1071-4c8f-a730-4042481bbc79.png)

![image](https://user-images.githubusercontent.com/71001536/170326679-184442d6-77f8-4aba-b525-9c47d8ad13e8.png)

![image](https://user-images.githubusercontent.com/71001536/170328867-3390f2ed-86c3-4450-93de-88de5ac5a2e2.png)

![image](https://user-images.githubusercontent.com/71001536/170329903-208a9956-591f-4358-81cc-36dabd1955d9.png)



![image](https://user-images.githubusercontent.com/71001536/170330038-2ad4a71d-3f75-43f4-afe6-bebb97bdbda7.png)

# Sonarcube 

![image](https://user-images.githubusercontent.com/71001536/170340195-11b04c21-6525-4549-9737-107ef7cba250.png)

![image](https://user-images.githubusercontent.com/71001536/170346015-cc4e2b27-3059-45da-9b5e-aa049debc59e.png)


Link to my [php-tod-repo](https://github.com/Damdev-95/php-todo)




