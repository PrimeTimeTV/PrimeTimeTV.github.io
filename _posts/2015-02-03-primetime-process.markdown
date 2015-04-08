---
layout: post
title:  "PrimeTime Development Process"
date:   2015-02-03 15:40:56
categories: primetime
---

Sprint
=======
PrimeTime sprint lenght is 2 weeks. Sprint will be started on Friday and complete on Thursday after two weeks.
For example: Sprint #1 is started at 1st Friday, it will be completed at 14th Thursday.

Sprint will be separated into two phases.

* Day 1 - 5, development phase. At the end of phase on Thursday, one build will be deployed to staging server.
* Day 8 - 14, testing phase, testing and bug fixing will be performed on this phase. And final build will be deployed on staging server at 14th day of sprint.

Developer will be assigned to next sprint stories or bug issues before Wednesday (13th day). Assignee _must_ estimate every story and bug issue implementation time. 

At the end of sprint on Thursday (14th day), sprint meeting must be setup. Sprint master will arrange the estimated issues based on developer estimation. Every story must be estimated and explained in this meeting. Total estimation time from all issues of each developer should not be longer than 5 days.

![Sprint time table](/images/primetime/sprint_time_table.png)

Kanban
------
Kanban board is mainly used in development process including coding, and testing. Our Kanban board will be splited into 4 columns with 6 story statues.

![Kanban](/images/primetime/kanban.png)

Version Control Process
=======================
We use Github as a version control repository. Every repository will have two main branches called "development" and "master". 

Development branch is branch where developers will fork a new feature branch from here and push it back when task is finished. This branch will be deployed to Development server when development phases is completed.

![Git Flow](/images/primetime/gitflow.png)

Master branch is the stable brach. Developers are not allowed to commit any code or artifacts to this branch. Only control master is allowed to merge code from "development" branch into "master" branch. This branch will be deployed to Production server.

Every story and bug issue must be implemented on feature branch. Developer will create a new branch from "development" branch. Branch name must be formatted as "{issue_id} subject" e.g. "API-123 add display field to content"

When implementation is finished, developer will merge "feature" branch into "development" branch. Developers are allowed to commit code as must as need. But only tested and working code is allowed to merged into "development" branch.

Development Process 
===================

1. Developer pick a card from sprint.
2. Developer will implement assigned stories and assigned bug issues on development phase (day 1 - 5). Code will be committed into feature branch.
3. Every committed code must have a valid comment. The comment format is "{issue_id} description" e.g. "API-123 add display field to content repository"
4. The service component must have unit test and instruction coverage more than 50% (This value will be increased to 80% at the end of year 2015)
5. The repository component may have unit test
6. The utilities and the other classes must have unit test with instruction coverage more than 50% (This value will be increased to 80% at the end of year 2015)
7. The REST interface must have JMeter test
8. When code is completed with unit test, developer will move a card to "resolved" stage. Code will be merged into "development" branch.
9. If bug is occurred, developer much fix bugs on story branch and re-merge into "development" branch again when it's completed.

![Unit Test Result](/images/primetime/unit_test_result.png)
![Coverage Test Result](/images/primetime/coverage_test_result.png)
![Integration Test Result](/images/primetime/integration_test_result.png)

Deployment Process
==================
* Development server will be deployed on half sprint and the end of sprint. Every developer has a permission to deploy code on development server.
* Tester and every 3rd parties will use this development server for development process.
* Unit test will be performed against development server every night. 
* Integration test or scenario test will be automatically run after deployment is completed. If testing result is not pass, deployer should restore previous build artifact to server within 1 hour.

Release Process
===============
* Once development server has been tested and approved by QA manager, the approved revision will be merged into "master" branch. 
* The production server will be deployed only when it is called by Product Manager and QA manager.
* Integration test or scenario test will be automatically run after deployment is completed. Human test may requires to make sure that all functionals are completed. If testing result is not pass, deployer should restore previous build artifact to server immediatly.
* Only successfully deployed revision will be tagged as release

![Working Environment](/images/primetime/working_environment.png)

Hot Fix Process
===============
Hot fix is critical issues we need to resolve it on Production server as soon as possible.

* Hot fix will be implemented on separated brach. The developer much create a hot fix branch from "master" branch.
* Hot fix branch may be tested on "development" branch before merge back into "master" branch
* Once issue has been resolved and tested, hot fix branch will be merged back to "master" branch and perform release process again

Code Quality
============
...

Code Convention
---------------
PrimeTime Java Style Convention is based on Google Style Guide https://google-styleguide.googlecode.com/svn/trunk/javaguide.html. Every code before committed or pushed to repository must be compiled with this standard.

CheckStyle
----------
In PrimeTime, we also use use CheckStyle as the style complier, every developer _must_ apply the CheckStyle format to their IDE.

The example of CheckStyle constraint

* JavaDoc must be presented on every Publish, Protected and Default methods and classes
* Name of class, method, parameter, variable must be compiled with PrimeTime Java Style Convention


![Checkstyle Warning](/images/primetime/checkstyle_warning.png)

IDE Compiler Warning
--------------------
PrimeTime also uses Java/IDE complier warning messages as a code quality standard. Every class must not have any warning or error present before push to repository.


![Eclipse IDE Warning](/images/primetime/eclipse_ide_warning.png)
