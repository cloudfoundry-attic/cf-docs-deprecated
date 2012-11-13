---
title: Hello World Spring MVC Project
description: Create a Spring MVC Hello World Application
tags:
    - Spring MVC
    - STS
    - Tomcat
    - Maven
---

## Introduction
* Spring's Web MVC framework is designed around a `DispatcherServlet` that dispatches requests to `handlers`, with configurable handler mappings, view resolution, locale and theme resolution as well as support for uploading files. The controller interface offers only a `ModelAndView handleRequest(request,response)` method. This could be used for application controllers, but you would rather have the included implementation hierarchy consisting of say, `AbstractController`, `AbstractCommandController` and `SimpleFormController`. For more details, visit the [springsource.org](http://static.springsource.org/spring/docs/2.0.x/reference/mvc.html) site.

	<img style="max-width:80%" src="/images/spring_tutorial/spring-mvc-flow.png" alt="spring-mvc-flow">

* This section provides details on how to get started with STS (Spring Tool Suite) and Spring MVC to create a Hello world Spring MVC application.

## Create a Spring MVC Template Project
1. Please open STS and navigate to the dashboard (**Help -> Dashboard**). From the dashboard, select **Spring Template Project**. You should see the window below.
  ![Spring MVC Template Project](/images/spring_tutorial/spring_template_project_mvc.png)
2. Choose **Spring MVC Project**.  STS will ask you to download some extra
elements if this your first time selecting this option.
  ![spring_template_project_mvc_download.png](/images/spring_tutorial/spring_template_project_mvc_download.png)
3. Give your project the name `expensereport` and specify your top-level package as `com.springsource.html5expense`
4. Click on **Finish** and STS will create the project.
5. Before running the project, right click on it and select **Run As -> Maven clean**

    ![maven_clean.png](/images/spring_tutorial/maven_clean.png)

6. Once Maven clean completes, select **Run As -> Maven install**. Dependencies will be downloaded from pom.xml.

    ![maven_install.png](/images/spring_tutorial/maven_install.png)

7. Now Select **Run As -> Maven build**

    ![maven_build.png](/images/spring_tutorial/maven_build.png)

8. Enter **tomcat:run** in Goals and click **Run**.

    ![maven_run.png](/images/spring_tutorial/maven_run.png)

## Check Point
*  Once the server has started, open your browser and go to the application url `http://localhost:8080/html5expense`. It will open the home page for the project which displays the Hello world message with the current server time. It should look like the screenshot below.

	![Hello World Output](/images/spring_tutorial/hello_world.png)

<a class="button-plain" style="padding: 3px 15px;" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-getting-started-with-STS.html">Prev</a> <a class="button-plain" style="padding: 3px 15px; float: right" href="/frameworks/java/spring/tutorials/springmvc-jpa-postgres/spring-expensereport-app-tutorial.html">Next</a>
