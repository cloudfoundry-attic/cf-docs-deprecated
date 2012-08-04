---
title: Play Java mysql App on Cloud Foundry
description: Play Java Application with mysql backend deployed on Cloud Foundry
tags:
    - play
    - java
    - mysql
    - tutorial
---

This is a guide for Java developers using Play framework and mysql database to build and 
deploy their apps and deploy on Cloud Foundry. It shows you how to set up and successfully 
deploy Play Java mysql applications to Cloud Foundry.


Before you get started, you need the following:

+	A [Cloud Foundry account](http://cloudfoundry.com/signup)

+	The [vmc](/tools/vmc/installing-vmc.html) Cloud Foundry command line tool

+	A [Play 2.0 ](http://www.playframework.org/documentation/2.0.2/Home) installation

## Introduction

In this tutorial we take a simple use case : 

[TodoList App]( http://www.playframework.org/documentation/2.0.2/JavaTodoList ) app described 
in the Play Framework website and show how to deploy it on Cloud Foundry.

## TodoList Play App

TodoList is used to create and manage tasks. 

![todolist-usecase.png](/images/play/todolist-usecase.png)

These tasks are created by the users and the data
is stored in underlying relational store.
The default tutorial stores tasks in the embedded database which comes with the Play framework.

Various components of the app can be found in [Todo List App Overview]( /frameworks/play/todolistjavaapp.html) document.

## Build and run the App Locally
The app can be run locally using the standard command from within the app folder

``` bash
$play run
```
The following screen appears the first time as the evolution script needs to be applied and
tables created

![play-local-java-mysql-applyscript.png](/images/screenshots/play/play-local-java-mysql-applyscript.png)

Once the user pressess the ApplyScript button the actual app home shows up

![play-local-java-mysql.png](/images/screenshots/play/play-local-java-mysql.png)


### Modifying the evolutions file for mysql

#### Comment / Delete 
We will need to comment out the following statements as they are not recognized by mysql

```  sql
create sequence task_seq
drop sequence task_seq
SET REFERENTIAL_INTEGRITY FALSE
SET REFERENTIAL_INTEGRITY TRUE
```

#### Modify 
We will modify the create statement to add auto increment functionality to id field.

``` sql
create table task (
  id                        bigint not null AUTO_INCREMENT,
  label                     varchar(255),
  constraint pk_task primary key (id))
;
```

## Deploy the App on Cloud Foundry
Deployment to Cloud Foundry can be done in two simple steps

+	Create a distribution
+	Push the distribution using `vmc push`

Commands below show the exact commands and output when the app is deployed 

``` bash
$ play dist
[info] Loading project definition from /Users/[username]/play/mysamples/todolist-java-mysql/project
[info] Set current project to todolist-java-mysql (in build file:/Users/rajdeepd/vmware/play/mysamples/todolist-java-mysql/)
[info] Packaging /Users/[username]/play/mysamples/todolist-java-mysql/target/scala-2.9.1/todolist-java-mysql_2.9.1-1.0-SNAPSHOT.jar ...
[info] Done packaging.

Your application is ready in /Users/[username]/play/mysamples/todolist-java-mysql/dist/todolist-java-mysql-1.0-SNAPSHOT.zip

[success] Total time: 3 s, completed Aug 2, 2012 11:11:55 PM
$ vmc push --path=dist/todolist-java-mysql-1.0-SNAPSHOT.zip 
Application Name: todolist-java-mysql
Detected a Play Framework Application, is this correct? [Yn]: Y
Application Deployed URL [todolist-java-mysql.cloudfoundry.com]: y
Memory reservation (128M, 256M, 512M, 1G, 2G) [256M]: 
How many instances? [1]: 1
Bind existing services to 'todolist-java-mysql'? [yN]: N
Create services to bind to 'todolist-java-mysql'? [yN]: y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 2
Specify the name of the service [mysql-2ea2b]: 
Create another? [yN]: N
Would you like to save this configuration? [yN]: y
Manifest written to manifest.yml.
Creating Application: OK
Creating Service [mysql-2ea2b]: OK
Binding Service [mysql-2ea2b]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (83K): OK   
Push Status: OK
Staging Application 'todolist-java-mysql': OK                                   
Starting Application 'todolist-java-mysql': OK                                  

```

Upon completion of deployment we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`

![play-cf-java-mysql.png](/images/screenshots/play/play-cf-java-mysql.png)

## Summary
In this tutorial we learnt how to build and deploy and basic Play Java app using mysql as the backend



 

