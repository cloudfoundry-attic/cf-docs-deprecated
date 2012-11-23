---
title: Hello World Spring MVC Project
description: Create Spring MVC Hello World Application
tags:
    - spring-mvc
    - sts
    - tomcat
    - maven
---

## Introduction
* This section provides details on how to get started with STS (Spring Tool Suite) and Spring MVC to create a Hello world Spring MVC application. The SpringSource Tool Suite (STS) is a free, Eclipse-based distribution. It is a super set of the Java EE distribution of Eclipse, and includes a slew of handy plugins pre-installed, including Maven support. Maven is a tool that lets you easily compile your code.

* To make sure that everything is easy to find in STS for this tutorial, use the **Window -> Reset** Perspective command.

    ![STS reset view](/images/spring_tutorial/sts-reset-view.png)

* If you're using STS you'll need to install the Cloud Foundry WTP (web tools project) connector. Go to the Help -> Eclipse Marketplace. Search for "CloudFoundry" and you'll find the "Cloud Foundry integration for Eclipse." Install it if you haven't already.

    ![STS reset view](/images/spring_tutorial/eclipse-marketplace.png)

* Once Cloud Foundry plugin installed, Go to servers, right click to add New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry, then click next.

    ![cloud foundry runtime](/images/spring_tutorial/cloud_foundry.png)

* Now enter your Cloud Foundry account information, choose **VMware Cloud Foundry - https://api.cloudfoundry.com** for the URL, and click **Finish**.

    ![cloud foundry account information.png](/images/spring_tutorial/cloud_foundry_account.png)

* Once you installed Cloud Foundry server, you can double click on the Cloud Foundry server to see what's going on in your account. The tab that presents has two sub tabs at the bottom: one called `Overview`, and another called `Applications`. You can click on an application to see the resources provisioned for the application, including which services (databases, message queues, etc) have been apportioned for it and how much RAM and CPU the application is taking.

   ![cf-editor](/images/screenshots/configuring-STS/cf_eclipse_cf_editor.png)

## Create Spring MVC Template Project
1. Please open STS and navigate to the dashboard (**Help -> Dashboard**). From the dashboard, select **Spring Template Project**. You should see the window below.
  ![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)
2. Choose **Spring MVC Project**.  STS will ask you to download some extra
elements if this your first time selecting this option.
  ![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)
3. Give your project the name `expensereport` and specify your top-level package as `com.springsource.html5expense`
4. Click on **Finish** and STS will create the project.

## Deploy on Local Server
1. Select project, drag it from `Project` panel and drop into your VMware vFabric tc server listed in the bottom left Server window.
2. Select VMware vFabric tc server and press **CTRL+ALT+R** to run the server.
3. Once server start is completed, open your browser and go to the application url `http://localhost:8080/html5expense` it will open the home page for the project which displays Hello world message with the current server time.

	![Hello World Output](/images/spring_tutorial/hello_world.png)

## Deploy on Cloud Foundry
1. Select project, drag it from `Project` panel and drop into your Cloud Foundry server listed in the bottom left Server window. It'll ask you to specify information about the application: what runtime, what language, etc. The defaults are good enough in this case, so just leave them as-is and click `Next`.

    ![spring-expensereport-project-deploy-step1](/images/spring_tutorial/project_deploy_step2.png)

2. Next, you'll be asked to specify the URL for the application as well as how much memory to allocate. Again, the defaults are fine in this case.

    ![spring-expensereport-project-deploy-step2](/images/spring_tutorial/project_deploy_step3.png)

3. The application will deploy, and - if you open the URL given in a browser window - you'll be greeted with a new application on Cloud Foundry!

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-getting-started-with-STS.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html">Next</a>
