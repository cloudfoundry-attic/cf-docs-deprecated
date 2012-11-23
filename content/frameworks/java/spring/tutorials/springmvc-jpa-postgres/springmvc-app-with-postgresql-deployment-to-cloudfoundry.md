---
title: Spring MVC Application with PostgreSQL Deployment to Cloud Foundry
description: Spring MVC App with PostgreSQL Deployment to Cloud Foundry
tags:
    - spring
    - STS
    - tomcat
    - maven
    - VMC
---

## Introduction
This section provides details on how to deploy a Spring Application to Cloud Foundry using STS and VMC.

## Prerequisites
Before you begin this tutorial, you should be done with:

1. Getting the Cloud Foundry plugin for STS installed.
2. Installing VMC.
3. Assuming previous exercise get completed.

## Subtopics

+ [Deploy Spring MVC App with PostgreSQL to Cloud Foundry using STS](#deploy-spring-mvc-app-with-postgresql-to-cloud-foundry-using-sts)
+ [Deploy Spring MVC App with PostgreSQL to Cloud Foundry using VMC](#deploy-spring-mvc-app-with-postgresql-to-cloud-foundry-using-vmc)

## Deploy Spring MVC App with PostgreSQL to Cloud Foundry using STS
1. Go to Servers view and right click to add a New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry and then click next. It will ask your account information, at which point you can choose **VMware Cloud Foundry - http://api.cloudfoundry.com** as the URL.

    ![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)

2. Once you have entered your Cloud Foundry account details, **Click validate** to make sure you have entered valid credentials.

    ![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)

3. Clicking **Next** will take you to **Add and Remove**. Now you can select your spring application. Once you have added your application, it will open a new window with the application name and type. This is where you can enter your application's name.

    ![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry_project_deploy.png)

    ![spring-expensereport-project-deploy.png](/images/spring_tutorial/project_deploy_step2.png)

4. Click **Next** to see your deployed URL(your application name+cloudfoundry.com) and memory reservation. By default, memory reservation is 512M. You can increase memory if required.

    ![spring-expensereport-project-deploy-step-2.png](/images/spring_tutorial/project_deploy_step3.png)

5. Click **Next** to see your Services selection.  Since we are using PostgreSQL in our application, we need to add a PostgreSQL service.

    ![service_selection.png](/images/spring_tutorial/service_selection.png)

    ![create_new_service.png](/images/spring_tutorial/create_new_service.png)

6. Now you can see your services in services selection window. Select PostgreSQL service and click finish. Now server starts to deploy our application to Cloud Foundry.
  ![spring-expensereport-login.png](/images/spring_tutorial/service_selection_1.png)

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_tutorial/deployed_application_in_cloud_foundry.png)

* To get the information about Tunneling services on Cloud Foundry please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/postgresql-dataservice-tunnel-on-cloudfoundry.html).

## Deploy Spring MVC App with PostgreSQL to Cloud Foundry using VMC
Deployment to Cloud Foundry using VMC can be done in two simple steps:

**1**. Create war file

**2**. Push the war file using `vmc`

Commands below show the exact commands and output when the app is deployed:

**Step 1**  Go to your project work space and give **mvn clean install** to create war. The war file will be generated in `target` folder

``` bash
$ mvn war:war
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building SpringMVC-ExpenseReport 1.0.0-BUILD-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-war-plugin:2.1.1:war (default-cli) @ html5expense ---
[INFO] Packaging webapp
[INFO] Assembling webapp [html5expense] in [/home/senthils/Code/From Home/SpringMVC-ExpenseReport/target/html5expense-1.0.0-BUILD-SNAPSHOT]
[INFO] Processing war project
[INFO] Copying webapp resources [/home/senthils/Code/From Home/SpringMVC-ExpenseReport/src/main/webapp]
[INFO] Webapp assembled in [380 msecs]
[INFO] Building war: /home/senthils/Code/From Home/SpringMVC-ExpenseReport/target/html5expense-1.0.0-BUILD-SNAPSHOT.war
[INFO] WEB-INF/web.xml already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.443s
[INFO] Finished at: Mon Sep 17 15:57:50 IST 2012
[INFO] Final Memory: 6M/124M
[INFO] ------------------------------------------------------------------------
```
**Step 2**  Now point vmc target to `http://api.cloudfoundry.com` and login to Cloud Foundry using your credentials. Once successfully logged into Cloud Foundry, give command **vmc push** and select deployment from current directory to *No* and give project target folder path in deployment path.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: ***********
Password: **********
Successfully logged into [http://api.cloudfoundry.com]

$ vmc push
Would you like to deploy from the current directory? [Yn]: N
Deployment path:/home/user/expensereport/target
Application Name: html5expense
Detected a Java SpringSource Spring Application, is this correct? [Yn]: Y
Application Deployed URL [html5expense.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [512M]: 128M
How many instances? [1]:
Bind existing services to 'html5expense'? [yN]:
Create services to bind to 'html5expense'? [yN]: Y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 3
Specify the name of the service [postgresql-94876]:
Create another? [yN]:
Would you like to save this configuration? [yN]:
Creating Application: OK
Creating Service [postgresql-94876]: OK
Binding Service [postgresql-94876]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (10K): OK
Push Status: OK
Staging Application 'html5expense': OK
Starting Application 'html5expense': OK
```

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_tutorial/deployed_application_in_cloud_foundry.png)


* To get the information about Tunneling services on Cloud Foundry please refer [here](/frameworks/java/spring/tutorials/springmvc-jpa-postgres/postgresql-dataservice-tunnel-on-cloudfoundry.html).
