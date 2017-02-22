# TDD Configuration Notes

### Goal:
To automate the process of evaluating Chef Cookbooks using Jenkins along chef and ruby tools, like Inspec, FoodCritic and Rubocop

### Assumptions, prerequisites:

There should be a working Jenkins server >= 2.32.2 with ChedDK installed. Also, it should have the SCM credentials configured

## Install Jenkins plugins

Delivery pipeline plugin
pipeline
Shared workspace
Warnings plugin


## Create Shared workspace for all the jobs

Manage Jenkins -> Configure System -> workspace sharing

Name:  TDDworkspace
Repository URL: https://github.com/douglax/TDDwChef.git

## Create SCM retrieval jobs

jenkinsserver:8080/newjob

Enter an item name: SCM_retrieval
Select Freestyle project, then OK

Configure the job with the following information:

Delivery pipeline Configuration
Stage name:  Stage 1 - Git pull
Task name: Code retrieval

Shared workspace: TDDworkspace
Source code management: git
Repository URL: https://github.com/douglax/TDDwChef.git
Credentials: Previously configured, click add if none exist
Build triggers:
Github hook trigger for SCM polling
Poll SCM: Schedule   * * * * *

Build environment
Add timestamps for console output
Create Delivary pipeline version

Save


## Create Inspec evaluation job

jenkinsserver:8080/newjob

Enter an item name: InspecAssessment
Select Freestyle project, then OK

Configure the job with the following information:

Delivery pipeline Configuration
Stage name:  Stage 2 - Inspec
Task name: Inspec Test


Shared workspace: TDDworkspace

Build triggers

Build after other projects are built ->  SCM_retrieval

Post-build action

Publish JUnit test report

Test report XMLs:   *.xml

Save
