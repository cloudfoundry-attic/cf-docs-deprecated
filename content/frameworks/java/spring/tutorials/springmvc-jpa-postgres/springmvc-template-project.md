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
* Spring's Web MVC framework is designed around a `DispatcherServlet` that dispatches requests to `handlers`, with configurable handler mappings, view resolution, locale and theme resolution as well as support for upload files. Controller interface, offering only a `ModelAndView handleRequest(request,response)` method. This could be used already for application controllers, but you would rather have the included implementation hierarchy consisting of say, `AbstractController`, `AbstractCommandController` and `SimpleFormController`. For more details visit the [springsource.org](http://static.springsource.org/spring/docs/2.0.x/reference/mvc.html) site.

	<img style="max-width:80%" src="/images/spring_tutorial/spring-mvc-flow.png" alt="spring-mvc-flow">
 
* This section provides details on how to get started with STS (Spring Tool Suite) and Spring MVC to create a Hello world Spring MVC application. The SpringSource Tool Suite (STS) is a free, Eclipse-based distribution. It is a super set of the Java EE distribution of Eclipse, and includes a slew of handy plugins pre-installed, including Maven support. Maven is a tool that lets you easily compile your code.

  You can get the instruction to install Cloud Foundry plugin for STS from [here](/tools/STS/configuring-STS.html).

  Once Cloud Foundry plugin installed, Go to servers, right click to add New Server. Now you can see Cloud Foundry under VMware. Select Cloud Foundry, then click next.Now enter your Cloud Foundry account information, choose **VMware Cloud Foundry - https://api.cloudfoundry.com** for the URL, and click **Finish**. Once you added Cloud Foundry run time, you can double click on the Cloud Foundry server to see what's going on in your account. The tab has two sub tabs at the bottom: one called `Overview`, and another called `Applications`. You can click on an application to see the resources provisioned for the application, including which services (databases, message queues, etc) have been apportioned for it and how much RAM and CPU the application is taking. Under instances one link is available(**Remote Systems View**) to access the deployed applications logs in Cloud Foundry. On clicking of it it will open Remote File Systems view panel like this.

## Create Spring MVC Template Project
1. Please open STS and navigate to the dashboard (**Help -> Dashboard**). From the dashboard, select **Spring Template Project**. You should see the window below.
  ![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)
2. Choose **Spring MVC Project**.  STS will ask you to download some extra
elements if this your first time selecting this option.
  ![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)
3. Give your project the name `expensereport` and specify your top-level package as `com.springsource.html5expense`
4. Click on **Finish** and STS will create the project.
5. Select project, drag it down and drop into your VMware vFabric tc server listed in the bottom left Server window.
6. Select VMware vFabric tc server and press **CTRL+ALT+R** to run the server.

## Check Point
*  Once server start is completed, open your browser and go to the application url `http://localhost:8080/html5expense` it will open the home page for the project which displays Hello world message with the current server time.

	![Hello World Output](/images/spring_tutorial/hello_world.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-getting-started-with-STS.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html">Next</a>
