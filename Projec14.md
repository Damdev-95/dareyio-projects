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
    }
}
```
* To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

* Click on the "Administration" button

* Navigate to the Ansible project and click on "Scan repository now"

![image](https://user-images.githubusercontent.com/71001536/169817817-0b65ebd4-8bcf-4163-a643-f9fe567e78e1.png)



