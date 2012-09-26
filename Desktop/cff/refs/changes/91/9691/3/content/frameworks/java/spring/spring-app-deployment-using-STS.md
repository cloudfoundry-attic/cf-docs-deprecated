---
title: STS to deploy Spring Application
description: Spring Application Deployment to Cloud Foundry using STS
tags:
    - spring
    - STS
    - tomcat
    - maven
---

## Introduction
This section provides details on how to push Spring Application into Cloud Foundry using STS.

## Prerequisites
Before you begin this tutorial, you should be familiar with:

1.  Cloud Foundry plugin for STS installed.

2.  Assuming previous exercise get completed.

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

[<==Prev      ](/frameworks/java/spring/expensereport-app-with-spring-security.html)              [        Next==>](/frameworks/java/spring/spring-app-deployment-using-VMC.html)
