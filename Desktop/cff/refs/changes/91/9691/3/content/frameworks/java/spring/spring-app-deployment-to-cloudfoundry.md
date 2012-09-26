---
title: VMC to deploy Spring Application
description: Spring Application Deployment to Cloud Foundry
tags:
    - spring
    - STS
    - tomcat
    - maven
    - VMC
---

## Introduction
This section provides details on how to push Spring Application into Cloud Foundry using STS and VMC.

## Prerequisites
Before you begin this tutorial, you should done with:

1.  Cloud Foundry plugin for STS installed.

2.  VMC tool installed.

3.  Assuming previous exercise get completed.

## Subtopics

+ [Deploy Spring Application into Cloud Foundry using STS](#programming-and-packaging-spring-applications)
+ [Deploy Spring Application into Cloud Foundry using VMC](#deploy-the-app-on-cloud-foundry-using-vmc)

## Deploy the App on Cloud Foundry Using STS
1.  Go to servers view,right click add New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry,then click next.It will ask your account information and Choose **VMware Cloud Foundry - http://api.cloudfoundry.com** from the URL.

	![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry.png)

2.  Now give your Cloud Foundry account information. **Click validate** and make sure you have entered  valid credentials.

	![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)

3.  Click next it will show you add and remove view.Now you can select our spring application. Once you added your application it will open new window with application name and application type. You can enter your application name.

	![spring-expensereport-login.png](/images/spring_tutorial/cloud_foundry_project_deploy.png) 

	![spring-expensereport-project-deploy.png](/images/spring_tutorial/project_deploy_step2.png)

4.  Click next you can see deployed URL(your application name+cloudfoundry.com) and memory reservation. By default memory reservation selected to 512M.You can increase memory if required.

	![spring-expensereport-project-deploy-step-2.png](/images/spring_tutorial/project_deploy_step3.png)

5.  Click next now it will ask for services selection. Since we are using postgresql in our application we need to add postgresql service. Click add services to server it will open service configuration window. Select type as **Postgre SQL database service(VFabric)** and give name to this service then click 

	![service_selection.png](/images/spring_tutorial/service_selection.png)

	![create_new_service.png](/images/spring_tutorial/create_new_service.png)

6.  Now you can see your services in services selection window. Select postgresql service and click finish. Now server starts to deploy our application into Cloud Foundry.

	![spring-expensereport-login.png](/images/spring_tutorial/service_selection_1.png)

## Check Point
Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`

![deployed_application_in_cloud_foundry.png](/images/spring_tutorial/deployed_application_in_cloud_foundry.png)


## Deploy the App on Cloud Foundry Using VMC
Deployment to Cloud Foundry using VMC can be done in two simple steps:

**1**.  Create war file

**2**.  Push the war file using `vmc`

Commands below show the exact commands and output when the app is deployed:

**Step 1**.  Go to your project work space and give **mvn war:war** to create war.

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
**Step 2**.  Now point vmc target to `http://api.cloudfoundry.cpm` and login to cloudfoundry using your credentials.

```bash
$ vmc target http://api.cloudfoundry.com
Successfully targeted to [http://api.cloudfoundry.com]

$ vmc login
Attempting login to [http://api.cloudfoundry.com]
Email: senthil.s@imaginea.com
Password: **********
Successfully logged into [http://api.cloudfoundry.com]

$ vmc push
Would you like to deploy from the current directory? [Yn]: /home/senthils/.rvm/gems/ruby-1.9.2-head/gems/interact-0.4.8/lib/interact/interactive.rb:569: warning: Insecure world writable dir /home/senthils/Downloads/springsource in PATH, mode 040777

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
Upon completion of deployment, we can go and visit the actual app at the URL `[app-name].cloudfoundry.com]`

![deployed_application_in_cloud_foundry.png](/images/spring_tutorial/deployed_application_in_cloud_foundry.png)


[![Prev](/images/spring_tutorial/prev_doc.png)](/frameworks/java/spring/expensereport-app-with-spring-security.html)
