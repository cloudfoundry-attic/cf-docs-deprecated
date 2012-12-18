---
title: Spring MVC Application with Redis Deployment to Cloud Foundry
description: Spring MVC Application with Redis Deployment to Cloud Foundry
tags:
    - spring
    - sts
    - tomcat
    - maven
    - vmc
    - redis
---

## Introduction
This section provides details on how to deploy a Spring Application to Cloud Foundry using STS and VMC.

## Prerequisites
Before you begin this tutorial, you should be done with:

1. Getting the Cloud Foundry plugin for STS installed.
2. Installing VMC.
3. Assuming previous exercise get completed.

## Subtopics

+ [Deploy Spring MVC App with Redis to Cloud Foundry using STS](#deploy-spring-mvc-app-with-redis-to-cloud-foundry-using-sts)
+ [Deploy Spring MVC App with Redis to Cloud Foundry using VMC](#deploy-spring-mvc-app-with-redis-to-cloud-foundry-using-vmc)

## Deploy Spring MVC App with Redis to Cloud Foundry using STS
1. Go to Servers view and right click to add a New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry and then click next. It will ask your account information, at which point you can choose **VMware Cloud Foundry - http://api.cloudfoundry.com** as the URL.
  ![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)
2. Once you have entered your Cloud Foundry account details, **Click validate** to make sure you have entered valid credentials.
  ![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)
3. Clicking **Next** will take you to **Add and Remove**. Now you can select your spring application. Once you have added your application, it will open a new window with the application name and type. This is where you can enter your application's name.
  ![spring-expensereport-deploy-cloudfoundry.png](/images/spring_redis_tutorial/cloud_foundry_project_deploy.png)
  ![spring-expensereport-project-deploy.png](/images/spring_redis_tutorial/project_deploy_step2.png)
4. Click **Next** to see your deployed URL(your application name+cloudfoundry.com) and memory reservation. By default, memory reservation is 512M. You can increase memory if required.

    ![spring-expensereport-project-deploy-step-2.png](/images/spring_redis_tutorial/project_deploy_step3.png)

5. Click **Next** to see your Services selection.  Since we are using Redis in our application, we need to add a Redis service.

    ![service_selection.png](/images/spring_redis_tutorial/service_selection.png)

    ![create_new_service.png](/images/spring_redis_tutorial/create_new_redis_service.png)

6. Now you can see your services in services selection window. Select Redis service and click finish. Now server starts to deploy our application to Cloud Foundry.

    ![spring-expensereport-login.png](/images/spring_redis_tutorial/redis_service_selection.png)

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_redis_tutorial/deployed_application_in_cloud_foundry.png)

## Deploy Spring MVC App with Redis to Cloud Foundry using VMC
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
[INFO] Building SpringMVC ExpenseReport Redis 1.0.0-BUILD-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-war-plugin:2.1.1:war (default-cli) @ springmvc-expensereport-redis ---
[INFO] Packaging webapp
[INFO] Assembling webapp [springmvc-expensereport-redis] in [/home/anand/Documents/vmware/cf/cf-tutorials/spring/springmvc-expensereport-redis/target/springmvc-expensereport-redis-1.0.0-BUILD-SNAPSHOT]
[INFO] Processing war project
[INFO] Copying webapp resources [/home/anand/Documents/vmware/cf/cf-tutorials/spring/springmvc-expensereport-redis/src/main/webapp]
[INFO] Webapp assembled in [138 msecs]
[INFO] Building war: /home/anand/Documents/vmware/cf/cf-tutorials/spring/springmvc-expensereport-redis/target/springmvc-expensereport-redis-1.0.0-BUILD-SNAPSHOT.war
[INFO] WEB-INF/web.xml already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.429s
[INFO] Finished at: Tue Dec 18 12:30:27 IST 2012
[INFO] Final Memory: 10M/164M
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
Application Name: springmvc-expensereport-redis
Detected a Java SpringSource Spring Application, is this correct? [Yn]: Y
Application Deployed URL [springmvc-expensereport-redis.cloudfoundry.com]:
Memory reservation (128M, 256M, 512M, 1G, 2G) [512M]:
How many instances? [1]: 1
Bind existing services to 'springmvc-expensereport-redis'? [yN]: N
Create services to bind to 'springmvc-expensereport-redis'? [yN]: y
1: mongodb
2: mysql
3: postgresql
4: rabbitmq
5: redis
What kind of service?: 5
Specify the name of the service [redis-ba917]: redis1
Create another? [yN]: N
Would you like to save this configuration? [yN]: N
Creating Application: OK
Creating Service [redis1]: OK
Binding Service [redis1]: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (5K): OK
Push Status: OK
Staging Application 'springmvc-expensereport-redis': OK
Starting Application 'springmvc-expensereport-redis': OK
```

## Check Point
1. Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`.
  ![deployed_application_in_cloud_foundry.png](/images/spring_redis_tutorial/deployed_application_in_cloud_foundry.png)

## Accessing Redis Service on Cloud Foundry through tunnel
To get the information about Tunneling services on Cloud Foundry please refer [here](/frameworks/java/spring/tutorials/springmvc-redis/redis-dataservice-tunnel-on-cloudfoundry.html).