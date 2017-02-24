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
Rake plugin


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

Test report XMLs:   '*.xml'

Save


## Create FoodCritic evaluation job

jenkinsserver:8080/configure

Compiler Warnings. Click the Add



jenkinsserver:8080/newjob

Enter an item name: FoodCriticAssessment
Select Freestyle project, then OK

Configure the job with the following information:

Delivery pipeline Configuration
Stage name:  Stage 3 - FoodCritic
Task name: FoodCritic Test


Shared workspace: TDDworkspace

Build triggers

Build after other projects are built ->  InspecAssessment

Post-build action

Scan for compiler Warnings

Add -> FoodCritic


## Create Rubocop evaluation job

 jenkins:8080/configure

Compiler Warnings. Click the Add

Name: Rubocop
Link name: https://github.com/bbatsov/rubocop
Trend report name: Ruby Lint Warnings
Regular Expression: ^([^:]+):(\d+):\d+: ([^:]): ([^:]+)$
Example Log Message: attributes/default.rb:21:78: C: Use %r only for regular expressions matching more than 1 '/' character.

Mapping Script:

'''
import hudson.plugins.warnings.parser.Warning

String fileName = matcher.group(1)
String lineNumber = matcher.group(2)
String category = matcher.group(3)
String message = matcher.group(4)

return new Warning(fileName, Integer.parseInt(lineNumber), "Ruby Lint Warnings", category, message);
'''

Create a Rakefile in the cookbook root

'''
# Rakefile

# -f correctness only fails the build on syntax violations.
# The warnings scan will still count and graph other violations.
desc 'Foodcritic correctness linting task'
task :foodcritic do
  sh 'foodcritic -f correctness .'
end

# --fail-level E will only fail Ruby errors.
# The warnings scan will still count and graph [W]arning and [C]op violations.
desc 'Rubocop linting task'
task :rubocop do
  sh 'rubocop --fail-level E'
end

# touchstone task
task jenkins: %w(rubocop foodcritic)
'''


jenkinsserver:8080/newjob

Enter an item name: RubocopAssessment
Select Freestyle project, then OK

Configure the job with the following information:

Delivery pipeline Configuration
Stage name:  Stage 4 - Rubocop
Task name: Rubocop Test


Shared workspace: TDDworkspace

Build triggers

Build after other projects are built ->  FoodCriticAssessment


Buil Step - Add

Invoke Rake
Tasks
rubocop


Post-build action

Scan for compiler Warnings

Add -> rubocop


## References:

https://blogs.sap.com/2016/03/18/running-foodcritic-on-jenkins-ci/
http://atomic-penguin.github.io/blog/2014/04/29/stupid-jenkins-and-chef-tricks-part-1-rubocop/
