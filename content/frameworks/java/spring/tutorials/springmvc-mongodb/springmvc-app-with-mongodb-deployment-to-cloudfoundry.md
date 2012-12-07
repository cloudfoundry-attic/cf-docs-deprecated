---
title: Spring MVC Application with MongoDB Deployment to Cloud Foundry
description: Spring MVC Application with MongoDB Deployment to Cloud Foundry
tags:
    - spring
    - sts
    - tomcat
    - maven
    - vmc
    - mongodb
---

## Introduction
This section provides details on how to deploy a Spring Application to Cloud Foundry using STS and VMC.

## Prerequisites
Before you begin this tutorial, you should be done with:

1. Getting the Cloud Foundry plugin for STS installed.
2. Installing VMC.
3. Assuming previous exercise get completed.

## Subtopics

+ [Deploy Spring MVC App with MongoDB to Cloud Foundry using STS](#deploy-spring-mvc-app-with-mongodb-to-cloud-foundry-using-sts)
+ [Deploy Spring MVC App with MongoDB to Cloud Foundry using VMC](#deploy-spring-mvc-app-with-mongodb-to-cloud-foundry-using-vmc)

## Deploy Spring MVC App with MongoDB to Cloud Foundry using STS
1. Go to Servers view and right click to add a New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry and then click next. It will ask your account information, at which point you can choose **VMware Cloud Foundry - http://api.cloudfoundry.com** as the URL.
  ![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)
2. Once you have entered your Cloud Foundry account details, **Click validate** to make sure you have entered valid credentials.
  ![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)
3. Clicking **Next** will take you to **Add and Remove**. Now you can select your spring application. Once you have added your application, it will open a new window with the application name and type. This is where you can enter your application's name.
  ![spring-expensereport-deploy-cloudfoundry.png](/images/spring_mongodb_tutorial/cloud_foundry_project_deploy.png)
  ![spring-expensereport-project-deploy.png](/images/spring_mongodb_tutorial/project_deploy_step2.png)
4. Click **Next** to see your deployed URL(your application name+cloudfoundry.com) and memory reservation. By default, memory reservation is 512M. You can increase memory if required.

    ![spring-expensereport-project-deploy-step-2.png](/images/spring_tutorial/project_deploy_step3.png)

5. Click **Next** to see your Services selection.  Since we are using MongoDB in our application, we need to add a MongoDB service.

    ![service_selection.png](/images/spring_tutorial/service_selection.png)

    ![create_new_service.png](/images/spring_mongodb_tutorial/create_new_mongo_service.png)

6. Now you can see your services in services selection window. Select MongoDB service and click finish. Now server starts to deploy our application to Cloud Foundry.

    ![spring-expensereport-login.png](/images/spring_mongodb_tutorial/mongo_service_selection.png)

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_mongodb_tutorial/deployed_application_in_cloud_foundry.png)

## Deploy Spring MVC App with MongoDB to Cloud Foundry using VMC
Deployment to Cloud Foundry using VMC can be done in two simple steps:

**1**. Create war file

**2**. Push the war file using `vmc`

Commands below show the exact commands and output when the app is deployed:

**Step 1**  Go to your project work space and give **mvn war:war** to create war.

``` bash
$ mvn war:war
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building SpringMVC ExpenseReport MongoDB 1.0.0-BUILD-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-war-plugin:2.1.1:war (default-cli) @ springmvc-expensereport-mongodb ---
[INFO] Packaging webapp
[INFO] Assembling webapp [springmvc-expensereport-mongodb] in [/home/anand/Documents/vmware/springmvc-expensereport-mongodb/target/springmvc-expensereport-mongodb-1.0.0-BUILD-SNAPSHOT]
[INFO] Processing war project
[INFO] Copying webapp resources [/home/anand/Documents/vmware/springmvc-expensereport-mongodb/src/main/webapp]
[INFO] Webapp assembled in [96 msecs]
[INFO] Building war: /home/anand/Documents/vmware/springmvc-expensereport-mongodb/target/springmvc-expensereport-mongodb-1.0.0-BUILD-SNAPSHOT.war
[INFO] WEB-INF/web.xml already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.741s
[INFO] Finished at: Thu Dec 06 16:29:02 IST 2012
[INFO] Final Memory: 11M/163M
[INFO] ------------------------------------------------------------------------
```
**Step 2**  Now point vmc target to `http://api.cloudfoundry.com` and login to Cloud Foundry using your credentials.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: *************
Password: **********
Successfully logged into [http://api.cloudfoundry.com]

$ vmc push
Would you like to deploy from the current directory? [Yn]: n
Deployment path: target
Application Name: springmvc-expensereport-mongodb
Detected a Java SpringSource Spring Application, is this correct? [Yn]: Y
Application Deployed URL [springmvc-expensereport-mongodb.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [512M]:
How many instances? [1]: 1
Create services to bind to 'springmvc-expensereport-mongodb'? [yN]: y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 1
Specify the name of the service [mongodb-b7562]: mongodb1
Create another? [yN]: N
Would you like to save this configuration? [yN]: N
Creating Application: OK
Creating Service [mongodb1]: OK
Binding Service [mongodb1]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (19K): OK
Push Status: OK
Staging Application 'springmvc-expensereport-mongodb': OK
Starting Application 'springmvc-expensereport-mongodb': OK
```

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_mongodb_tutorial/deployed_application_in_cloud_foundry.png)


## Accessing MongoDB Service on Cloud Foundry through tunnel
To get the information about Tunneling services on Cloud Foundry please refer [here](/frameworks/java/spring/tutorials/springmvc-mongodb/mongodb-dataservice-tunnel-on-cloudfoundry.html).